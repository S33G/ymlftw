# Integrations

How to wire `.agents/` context into each AI coding tool.

---

## OpenCode

Create `AGENTS.md` at your repo root:

```markdown
## Agent Context

Read and merge all files in `.agents/*.yml` at the start of every session.
Treat the merged object as your authoritative project context.
Keys from later-loaded files override earlier ones.
```

Or use `.opencode/rules/` for persistent injection:

```bash
mkdir -p .opencode/rules
```

`.opencode/rules/context.md`:
```markdown
---
alwaysApply: true
---

On session start: read and deep-merge all `.agents/*.yml` files.
Use the merged context for all decisions about stack, conventions, team, and deployment.
```

---

## Claude Code (claude.ai / claude CLI)

Create `CLAUDE.md` at repo root:

```markdown
## Project Context

At the start of every session, read and merge all `.agents/*.yml` files.
These files are your authoritative source of truth for this project.

Key files:
- `.agents/agents.yml` — service identity and MCP config
- `.agents/stack.yml` — languages, frameworks, infra
- `.agents/conventions.yml` — coding rules and patterns
- `.agents/team.yml` — team contacts
- `.agents/deployment.yml` — environments and CI/CD
```

---

## Cursor

Create `.cursor/rules/agents.mdc`:

```
---
alwaysApply: true
---

Read and merge all `.agents/*.yml` files on session start.
Use merged context as authoritative project data.
```

Or add to `.cursorrules` (legacy):
```
On session start: read and merge all files in .agents/*.yml as structured project context.
```

---

## GitHub Copilot

Create `.github/copilot-instructions.md`:

```markdown
## Project Context

Merge and apply all `.agents/*.yml` files as structured project context.
These are authoritative for stack, conventions, team, and deployment info.
```

---

## Any other tool

The pattern is always the same: instruct the agent to read `.agents/*.yml` at startup. The exact file varies by tool — the convention doesn't.

For tools with no persistent instruction file, paste this into your first message:
```
Before we start: read and merge all .agents/*.yml files as my project context.
```

---

## MCP (Model Context Protocol)

If your `agents.yml` contains an `mcp` section, agents that support MCP (OpenCode, Claude Code) can auto-configure servers from it.

Example in `agents.yml`:
```yaml
agent:
  mcp:
    gitlab:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-gitlab"]
      env:
        GITLAB_TOKEN: "${GITLAB_TOKEN}"
        GITLAB_URL: "${GITLAB_URL}"
```

For OpenCode, this maps directly to `.opencode/config.json` `mcpServers` format.
For Claude Code, map to `.claude/settings.json` `mcpServers`.
