[← Back to index](README.md)

# Summarizing a .docx File — Claude API vs. Amazon Bedrock

Two ways to reach Claude, two different file-handling rules. The task is the same for both: **upload a `.docx` and ask Claude to summarize it.**

> **Key difference up front:**
> - **Claude (Anthropic) API** — the `document` block accepts **PDF, plain text, and images**, but **not `.docx`**. So you extract the text from the `.docx` first and send it as text.
> - **Amazon Bedrock (Converse API)** — natively accepts a `.docx` as a `document` block. You send the **raw bytes** and Bedrock parses it for you.

## Setup
```bash
pip install anthropic python-docx boto3
export ANTHROPIC_API_KEY="sk-ant-..."   # for the Claude API example
# Bedrock uses your AWS credentials (aws configure / env vars / IAM role)
```

---

## 1. Claude (Anthropic) API — extract text, then summarize

```python
from anthropic import Anthropic
from docx import Document   # python-docx

# 1. Read the .docx locally and pull out its text
#    (the Anthropic API can't ingest .docx directly)
def docx_to_text(path: str) -> str:
    doc = Document(path)
    return "\n".join(p.text for p in doc.paragraphs)

document_text = docx_to_text("report.docx")

# 2. Send that text with a summarize prompt
client = Anthropic()   # reads ANTHROPIC_API_KEY from the environment
response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": f"Summarize this document in 3 bullet points:\n\n{document_text}",
        }
    ],
)

# 3. Read the reply
print(response.content[0].text)
```

**What's happening:**
- `python-docx` opens the file and reads each paragraph's text. This is the "upload" step — you're turning the file into text the API accepts.
- The text goes straight into the user message alongside the instruction.
- `response.content[0].text` is Claude's summary.

> **Alternative:** if you'd rather not extract text, convert the `.docx` to a **PDF** first and send it as a `document` block (PDF *is* supported) — either base64-inline or via the Files API (`client.beta.files.upload`). Text extraction is simpler for a plain document; PDF preserves layout/tables.

---

## 2. Amazon Bedrock (Converse API) — upload the .docx directly

```python
import boto3

# 1. Read the raw bytes of the .docx (no text extraction needed)
with open("report.docx", "rb") as f:
    docx_bytes = f.read()

# 2. Bedrock runtime client (uses your AWS credentials + region)
bedrock = boto3.client("bedrock-runtime", region_name="us-east-1")

# 3. Send the file as a `document` block + a text prompt
response = bedrock.converse(
    # Exact ID varies — check your Bedrock console. Cross-region
    # inference profiles are usually prefixed, e.g. "us.anthropic.claude-sonnet-5".
    modelId="anthropic.claude-sonnet-5",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "document": {
                        "format": "docx",            # Bedrock parses it natively
                        "name": "report",            # letters/numbers/spaces only
                        "source": {"bytes": docx_bytes},
                    }
                },
                {"text": "Summarize this document in 3 bullet points."},
            ],
        }
    ],
)

# 4. Read the reply
print(response["output"]["message"]["content"][0]["text"])
```

**What's happening:**
- No `python-docx` — Bedrock's Converse API accepts `.docx` (also `pdf`, `csv`, `xlsx`, `html`, `txt`, `md`) as a document block, so you pass the raw bytes.
- `format` tells Bedrock how to parse; `name` is a label (keep it simple — alphanumeric and spaces).
- Content is a **list of blocks**: the document, then the instruction text.
- The response is a plain dict (boto3), so you index into it rather than using attributes.

---

## 3. Amazon Bedrock — point at a file already in S3

Instead of reading bytes yourself, you can hand Bedrock an **S3 location** and let it fetch the file. Handy when the document already lives in a bucket (uploads, pipelines) and you don't want to download it first.

```python
import boto3

bedrock = boto3.client("bedrock-runtime", region_name="us-east-1")

response = bedrock.converse(
    modelId="anthropic.claude-sonnet-5",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "document": {
                        "format": "docx",
                        "name": "report",
                        "source": {
                            "s3Location": {
                                "uri": "s3://my-bucket/reports/report.docx",
                                # bucketOwner is optional but recommended for
                                # cross-account safety (the 12-digit account ID)
                                "bucketOwner": "123456789012",
                            }
                        },
                    }
                },
                {"text": "Summarize this document in 3 bullet points."},
            ],
        }
    ],
)

print(response["output"]["message"]["content"][0]["text"])
```

**What's happening:**
- The only change from example 2 is the document **`source`**: `s3Location` (a `uri` + optional `bucketOwner`) instead of inline `bytes`.
- Bedrock reads the object from S3 itself — nothing is downloaded into your process.
- **Permissions:** the bucket must be in the **same region** as the Bedrock call, and the IAM identity making the request needs `s3:GetObject` on that object (Bedrock uses your caller's credentials to fetch it).

> **bytes vs. s3Location — which to use?**
> - `bytes` — simplest; the file is local or small, or already in memory.
> - `s3Location` — the file already lives in S3, is large, or is produced by an upstream pipeline. Avoids the download-then-reupload round trip.

---

## Side-by-side

| | Claude (Anthropic) API | Amazon Bedrock (Converse) |
|---|---|---|
| SDK | `anthropic` | `boto3` (`bedrock-runtime`) |
| Auth | `ANTHROPIC_API_KEY` | AWS credentials + region |
| `.docx` support | ❌ not native — extract text first | ✅ native — send raw bytes |
| Response shape | object (`response.content[0].text`) | dict (`response["output"]...`) |

> **On model IDs:** Anthropic uses bare IDs (`claude-sonnet-5`); Bedrock prefixes them (`anthropic.claude-sonnet-5`) and often needs a region-scoped **inference profile** ID (`us.anthropic....`). Always confirm the exact ID in your AWS console.
>
> **Note:** There's also an Anthropic-SDK path to Bedrock (`AnthropicBedrockMantle`) that mirrors the Claude API surface — but because it uses the same Messages shape, it has the **same** no-native-`.docx` rule as example 1. Use the boto3 Converse API above when you specifically want to hand Bedrock the file directly.
