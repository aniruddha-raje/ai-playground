[← Back to index](README.md)

# LangGraph

**What it is:** An open-source library (from the LangChain team) for building **stateful, multi-step** LLM apps as a **graph**. You define nodes (steps) and edges (what runs next). It's built for agents and loops, where LangChain chains are mostly straight lines.

**Key idea:** A shared **state** object flows through the graph. Each node reads the state, does work, and returns updates to it. Edges decide the path — including loops and branches a plain chain can't express.

## Setup
```bash
pip install langgraph langchain-anthropic
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## Example 1 — A minimal two-node graph

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

# 1. The shared state that flows between nodes
class State(TypedDict):
    text: str

# 2. Each node takes the state and returns fields to update
def make_uppercase(state: State):
    return {"text": state["text"].upper()}

def add_excitement(state: State):
    return {"text": state["text"] + "!!!"}

# 3. Build the graph
builder = StateGraph(State)
builder.add_node("upper", make_uppercase)
builder.add_node("excite", add_excitement)

# 4. Wire the flow: START -> upper -> excite -> END
builder.add_edge(START, "upper")
builder.add_edge("upper", "excite")
builder.add_edge("excite", END)

graph = builder.compile()

# 5. Run it
print(graph.invoke({"text": "hello"}))   # {'text': 'HELLO!!!'}
```

**What's happening, step by step:**
- `State` (a `TypedDict`) defines what data moves through the graph.
- Each **node** is a function: it gets the current state and returns only the fields it wants to change. LangGraph merges those updates in.
- `START` and `END` are built-in markers for where the graph begins and finishes.
- `add_edge(a, b)` means "after a, run b." `.compile()` freezes it into a runnable graph.

---

## Example 2 — A tool-using agent (the prebuilt way)

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent

# 1. A tool the agent can call
@tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"It's always sunny in {city}."

# 2. The model
model = ChatAnthropic(model="claude-sonnet-5", max_tokens=1024)

# 3. Build a ready-made agent: model + tools, loop included
agent = create_react_agent(model, tools=[get_weather])

# 4. Ask it something that needs the tool
result = agent.invoke(
    {"messages": [{"role": "user", "content": "What's the weather in Paris?"}]}
)

# 5. The final answer is the last message
print(result["messages"][-1].content)
```

**What's happening, step by step:**
- `create_react_agent` builds a graph that automates the agent loop for you: **call model → if it asks for a tool, run the tool → feed the result back → repeat until Claude gives a final answer.**
- In Example 2 of [LangChain](05-langchain.md) you ran that loop by hand; here LangGraph runs it as a graph with a built-in cycle.
- State here is the running list of `messages`. Each turn (model reply, tool result) gets appended.
- `result["messages"][-1]` is the last message — Claude's final answer after any tool calls.

---

## LangChain vs. LangGraph — quick contrast
- **LangChain** → best for straight-line pipelines (`prompt | model | parser`).
- **LangGraph** → best when you need **loops, branches, or memory across steps** — i.e. agents that decide their own next move.
