[← Back to index](README.md)

# LLM Fundamentals

Core ideas behind how a language model reads text.

## Tokens
- A **token** is a chunk of text (roughly a word or part of a word). Models read and generate text one token at a time.

## Embedding
- A list (vector) of numbers that represents a token's **meaning**, not just its spelling.
- Similar meanings end up with similar number patterns, which is how the model relates words to each other.

## Contextualization
- Choosing the right meaning of a token based on the **surrounding words/sentence**, since one word can have multiple meanings.
- Example: "bank" in *river bank* vs *savings bank* — same token, different meaning decided by context.
