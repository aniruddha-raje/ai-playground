[← Back to index](README.md)

# Claude Products & Project Config

The different ways to use Claude, and how projects carry context.

## Products
- **Claude AI** — the web chat at claude.ai. Chat only.
- **Claude Desktop** — the desktop app. Chat, with support for connectors/MCP.
- **Claude Code** — brings your codebase into agentic flows (edit, run, test code).
- **Cowork** — you grant file-system access; it can move and organize directories. Destructive actions need permission first.
- **Projects** — like Chat, but you import context (files/knowledge) that Claude analyzes and reuses across the conversation.
- **Artifacts** — self-contained HTML pages you can share via a Claude URL.

## Project context files
- **CLAUDE.md** — a project's context/instructions file. Generate it with the **/init** command.
- A **CLAUDE.md** can also live at the user/system level (`~/.claude/`) for **personal preferences that apply across all projects**.

> Rough rule: **CLAUDE.md** = context and preferences · **Skills** = reusable procedures · **Hooks** = enforced rules.
