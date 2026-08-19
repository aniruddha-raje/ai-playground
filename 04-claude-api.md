[← Back to index](README.md)

# Claude API

How you talk to Claude programmatically.

## Endpoint
```
POST https://api.anthropic.com/v1/messages
```

## Request parameters
- **API key** — your secret credential, sent in the `x-api-key` header (not in the body).
- **model** — which Claude model to use (e.g. `claude-opus-4-8`).
- **max_tokens** — the maximum number of tokens Claude may generate in its reply.
- **messages** — the conversation so far, a list of objects:
  ```json
  [{ "role": "user", "content": "Hello" }]
  ```
- **system** — top-level parameter for tone, persona, and instructions on how to answer.

> Note: `system` is its **own** parameter, not a message inside the `messages` list.

## Message roles
- **user** — a message from you/the person.
- **assistant** — a message from Claude. Include past assistant replies to continue a conversation.

## Response
- **id** — unique identifier for the response.
- **content** — a list of blocks; text replies are in a block's `text` field.
- **usage** — token counts (`input_tokens`, `output_tokens`).
- **stop_reason** — why Claude stopped: `end_turn`, `max_tokens`, `stop_sequence`, or `tool_use`.
