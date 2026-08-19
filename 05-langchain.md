[← Back to index](README.md)

# LangChain

**What it is:** An open-source framework for building apps on top of LLMs. It gives you reusable building blocks — models, prompts, output parsers, tools, retrievers — that you connect into a pipeline (a "chain").

**Key idea (LCEL):** LangChain Expression Language lets you pipe components with `|`, like Unix pipes: `prompt | model | parser`. Data flows left to right.

## Setup
```bash
pip install langchain langchain-anthropic
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## Example 1 — A simple prompt → model → text chain

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 1. The model wrapper (talks to Claude for you)
model = ChatAnthropic(model="claude-sonnet-5", max_tokens=1024)

# 2. A reusable prompt with a {topic} placeholder
prompt = ChatPromptTemplate.from_template(
    "Explain {topic} in one simple sentence."
)

# 3. Turns the model's message object into a plain string
parser = StrOutputParser()

# 4. Wire them into a chain with the | operator
chain = prompt | model | parser

# 5. Run it — the dict fills the {topic} placeholder
print(chain.invoke({"topic": "embeddings"}))
```

**What's happening, step by step:**
- `ChatAnthropic` is LangChain's adapter for Claude — you set the model once and reuse it.
- `ChatPromptTemplate` keeps prompt text separate from data. `{topic}` is filled in at run time.
- `StrOutputParser` extracts just the text (otherwise you'd get a message object with metadata).
- `prompt | model | parser` builds the pipeline; `.invoke()` sends data through it.

---

## Example 2 — Giving Claude a tool to call

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool

# 1. Define a tool — the docstring tells Claude what it does
@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers together."""
    return a * b

# 2. Tell the model which tools it may use
model = ChatAnthropic(model="claude-sonnet-5", max_tokens=1024)
model_with_tools = model.bind_tools([multiply])

# 3. Ask a question that needs the tool
response = model_with_tools.invoke("What is 12 times 8?")

# 4. Claude replies with a tool request, not the answer yet
print(response.tool_calls)   # [{'name': 'multiply', 'args': {'a': 12, 'b': 8}, ...}]

# 5. You run the tool with the args Claude chose
if response.tool_calls:
    call = response.tool_calls[0]
    result = multiply.invoke(call["args"])
    print(result)            # 96
```

**What's happening, step by step:**
- `@tool` turns a normal Python function into something Claude can call. The **docstring is the description** Claude reads to decide when to use it.
- `bind_tools([...])` attaches the tool list to the model.
- Claude doesn't run the tool itself — it returns a `tool_calls` list saying *which* tool and *what arguments*. **Your code executes it.**
- This "model decides, you execute" loop is the foundation of agents (see [LangGraph](06-langgraph.md), which automates the loop).
