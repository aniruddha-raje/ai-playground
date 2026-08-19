# Claude Study Notes

A set of short, plain-language notes on Claude — from core LLM concepts to the API and the Claude Code tooling.

## Contents

| # | Topic | What's inside |
|---|-------|---------------|
| 1 | [LLM Fundamentals](01-llm-fundamentals.md) | Embeddings, contextualization — how models turn words into meaning |
| 2 | [Claude Products](02-claude-products.md) | Claude AI, Desktop, Code, Cowork, Projects, Artifacts, CLAUDE.md |
| 3 | [Claude Code](03-claude-code.md) | Tools, permission modes, subagents, slash commands, skills, hooks |
| 4 | [Claude API](04-claude-api.md) | The Messages endpoint, request parameters, response shape, message roles |
| 5 | [LangChain](05-langchain.md) | Open-source LLM framework — chains, prompts, tools (2 code examples) |
| 6 | [LangGraph](06-langgraph.md) | Stateful, graph-based agents and loops (2 code examples) |
| 7 | [MCP Server](07-mcp-server.md) | Building an MCP server — the standard tool structure (input, params, output) |

> These notes favor **one- or two-line explanations**. They're a quick reference, not full documentation.

## Running the code examples

```bash
pip install -r requirements.txt
export ANTHROPIC_API_KEY="sk-ant-..."
```

See [requirements.txt](requirements.txt) for the full dependency list.
