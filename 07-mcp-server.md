[← Back to index](README.md)

# MCP Server

**What it is:** MCP (Model Context Protocol) is an open standard for exposing **tools, data, and prompts** to an LLM app (like Claude Desktop or Claude Code) in a uniform way. You write a **server**; the LLM app is the **client** that connects to it and calls your tools.

**Why it matters:** Instead of hard-coding a tool into one app, you build it once as an MCP server and any MCP-compatible client can use it.

## Setup
```bash
pip install "mcp[cli]"
```

---

## The standard tool structure

Every MCP tool has three well-defined parts. The SDK builds all of them automatically from your function's **type hints** and **docstring**:

| Part | Comes from | Purpose |
|------|-----------|---------|
| **Input / parameters** | Function arguments + type hints | The named, typed inputs the client must send (becomes a JSON schema) |
| **Description** | The docstring | Tells the LLM *when* and *how* to use the tool |
| **Output** | The return type + returned value | What the tool sends back to the client |

---

## Example — a minimal MCP server with one tool

```python
from mcp.server.fastmcp import FastMCP

# 1. Create the server (the name identifies it to clients)
mcp = FastMCP("calculator")

# 2. Define a tool. This ONE function specifies all three parts:
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers and return the sum."""
    #    ^input params      ^output type   ^description (docstring)
    return a + b            # <- the output value

# 3. Start the server
if __name__ == "__main__":
    mcp.run()
```

**How the three parts map to the code:**
- **Input / parameters:** `a: int, b: int`. The type hints (`int`) let the SDK generate an input schema, so the client knows it must send two integers named `a` and `b`.
- **Description:** the docstring `"Add two numbers and return the sum."` — this is what the LLM reads to decide when to call `add`.
- **Output:** the return type `-> int` describes the result; `return a + b` is the actual value sent back.

That's the full contract — no separate schema file needed. The decorator `@mcp.tool()` registers the function so clients can discover and call it.

---

## Adding a second tool (richer types)

```python
@mcp.tool()
def greet(name: str, times: int = 1) -> str:
    """Return a greeting for `name`, repeated `times` times."""
    return ("Hello, " + name + "! ") * times
```

- `name: str` is **required** (no default).
- `times: int = 1` is **optional** — the default makes it optional in the generated schema.
- `-> str` declares the output is text.

---

## Connecting it to a client

Run the server, then register it with an MCP client (e.g. Claude Desktop's config or Claude Code). A typical entry:

```json
{
  "mcpServers": {
    "calculator": {
      "command": "python",
      "args": ["/path/to/your/server.py"]
    }
  }
}
```

Once connected, the client lists your tools (from the docstrings), and the LLM can call `add` / `greet` with validated arguments — matching the "model decides, host executes" pattern from [LangChain](05-langchain.md) and [LangGraph](06-langgraph.md).

> **Note:** This example uses **stdio** transport (the client launches your script and talks over standard input/output) — the simplest setup for local tools. MCP also supports HTTP-based transports for remote servers.
