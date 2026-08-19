[← Back to index](README.md)

# Claude Code

The agentic CLI/app that works inside your codebase.

## Tools
Actions Claude can take on your behalf, such as:
- **Read** — read a file.
- **Grep** — search file contents.
- **Bash** — run shell commands. **BashOutput** — read output from a background command.
- **WebSearch** — search the web.
- **Skill** — invoke a packaged skill (see below).
- **MCP** — call tools exposed by connected MCP servers.

## Permission modes
Control how much Claude can do without asking:
- **Manual (default/normal)** — asks before each action.
- **Accept Edits** — auto-approves file edits, still asks for riskier actions.
- **Auto (bypass permissions)** — runs without asking. Use with care.
- **Plan** — read-only; Claude researches and proposes a plan without changing anything.

## Subagents
- Separate helper agents for smaller tasks, so the main agent's context/memory isn't overused.
- Useful for **unbiased reviews** — a spawned reviewer doesn't carry the main agent's full context.
- Example: "Spawn a subagent code reviewer for the changes we did last time."

## Useful slash commands
- **/compact** — compress the context window as it approaches the limit.
- **/context** — view details of your current context usage.
- **/init** — generate a `CLAUDE.md` file documenting the project.

## Skills (`.claude/`)
- Packaged instructions: coding standards, processes to follow, workflows.
- **Load only when needed** (e.g. matching your commit style when opening a PR), which saves context.
- Example: a commit/push/PR skill handles committing, pushing, and PR creation in one step instead of doing each manually.

## Hooks (`.claude/`)
- **Mandatory rules** the harness runs automatically around tool calls — not optional like skills.
- Example: run Prettier after any file edit.
