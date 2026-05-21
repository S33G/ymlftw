# ymlftw

> **Structured context for AI agents — distributed across your entire company.**

A lightweight convention for putting a `.agents/` directory in every repo. Each file is a **YAML context slice** — structured, mergeable, and consumed by any AI coding agent (OpenCode, Claude Code, Cursor, Copilot, etc.).

---

## Why not Markdown?

Everyone reaches for `CURSOR_RULES.md` or `CLAUDE.md` as a big wall of prose. It works, but it has real problems:

| | Markdown | YAML |
|---|---|---|
| Structure | Implied by headers/bullets | Explicit keys and types |
| Merging | Manual copy-paste | Deep-merge trivially |
| Agent parsing | "Interpret the prose" | Read the key |
| Validation | None | JSON Schema compatible |
| Queryable in CI | Regex hacks | `yq`, `jq` natively |
| Token cost | High (formatting overhead) | Low (data-dense) |
| Consistency | Author-dependent | Schema-enforced |
| Missing data | Silent gap in prose | Explicit `null` or missing key |

Prose forces agents to hallucinate structure. YAML gives agents data they can reason about deterministically — "the `stack.runtime` key is `node@22`" is unambiguous in a way that "we use Node 22" buried in a paragraph is not.

Markdown is great for *human documentation*. YAML is better for *machine consumption*.

---

## FAQ

### Does YAML help if the agent does not know which key to read?

Not by itself. If an agent has no goal, it may still scan the full context. YAML helps once the task is known because the agent can jump to stable paths instead of interpreting prose.

For example:

| Task | Useful YAML path |
|---|---|
| Run tests | `deployment.pipeline.test_command` |
| Follow repo conventions | `conventions.patterns.preferred` |
| Check the runtime | `stack.runtime` |
| Find the team contact | `team.contact` |

The value is not that YAML guesses intent. The value is that intent can map to a predictable key.

### Why does validation matter if the source could be Markdown?

Markdown can be reviewed by humans, but it is hard to validate automatically. YAML can be checked by schema, CI, or simple tooling before an agent consumes it.

Validation catches things like missing required fields, wrong types, invalid enum values, broken commands, and inconsistent structure across repos. That matters when the context drives agent behavior, CI setup, dashboards, or repo-wide automation.

### Is YAML actually cheaper in tokens?

Yes, especially when the same facts are repeated across many repos. The savings come from both fewer words and less interpretation.

Markdown:

```md
## Runtime and Tooling

This project uses Node.js version 22. The source code is written in TypeScript version 5.4. Package management is done with pnpm version 9.

For testing, use Vitest for unit tests and Playwright for end-to-end tests. The expected coverage threshold is 80 percent.
```

YAML:

```yaml
stack:
  runtime: node@22
  language: typescript@5.4
  package_manager: pnpm@9
  testing:
    unit: vitest
    e2e: playwright
    coverage_threshold: 80
```

The YAML version removes filler words and gives each fact a stable path. Agents do not need to decide whether "testing is done with Vitest" means current, preferred, required, or historical. `stack.testing.unit: vitest` is direct.

### How does YAML expose missing data?

YAML does not know what is missing unless there is an expected shape. The advantage is that omissions become machine-detectable when you use a schema or shared convention.

It also lets authors distinguish between unknown and intentionally empty values:

```yaml
team:
  members:
    - name: Bob Smith
      slack: null
```

Here, `slack: null` says the Slack handle is intentionally absent or unavailable. If the `slack` key is missing entirely, tooling can treat that as unknown, incomplete, or not modeled yet.

### What will make people adopt this?

Adoption will come from solving a repeated agent failure, not from the file format alone. Good first targets are problems teams already feel: agents using the wrong test command, ignoring repo conventions, missing deploy rules, or guessing service ownership.

YAML is the mechanism. The pitch is stronger when it shows a concrete before-and-after where structured context prevents a real mistake.

---

## How it works

```
your-repo/
└── .agents/
    ├── agents.yml       # service/agent identity
    ├── team.yml         # people and contacts
    ├── stack.yml        # technologies and versions
    ├── conventions.yml  # coding rules and patterns
    ├── deployment.yml   # environments and CI/CD
    └── gitlab.yml       # repo-specific VCS config
```

At session start, an AI agent **reads and merges all `.agents/*.yml` files** into a single context object. Files are independent — add, remove, or override any slice without touching others.

**Merging is just deep merge.** Later files override earlier ones. This means you can have org-level defaults and repo-level overrides — the repo wins.

---

## Quick Start

**1. Copy `.agents/` into your repo:**
```bash
cp -r .agents/ /path/to/your-repo/
```

**2. Edit each file for your project.**

**3. Add the agent integration file for your tool** (see [`docs/integrations.md`](docs/integrations.md)).

That's it.

---

## File reference

| File | Purpose |
|---|---|
| `agents.yml` | Service name, URLs, dashboards, linked agents |
| `team.yml` | Team members, roles, contacts, timezones |
| `stack.yml` | Languages, frameworks, databases, infrastructure |
| `conventions.yml` | Code style, patterns, what to avoid |
| `deployment.yml` | Environments, CI/CD pipelines, deploy process |
| `gitlab.yml` | Branch strategy, MR templates, pipeline notes |

Files are **optional**. Use only what's relevant.

---

## Extending

Add any key to any file — agents consume the full merged object. There is no enforced schema (though one is suggested in [`docs/schema.md`](docs/schema.md)).

Examples of useful additions:

```yaml
# .agents/context.yml — domain knowledge
context:
  domain: "payments processing"
  glossary:
    idempotency_key: "unique string per transaction to prevent duplicate charges"
    settlement: "process of moving funds from pending to available"
  invariants:
    - "Never mutate a completed transaction"
    - "All monetary values are integers (cents)"
```

```yaml
# .agents/runbooks.yml — operational knowledge
runbooks:
  on_call:
    - "Check Datadog dashboard first"
    - "If DB lag > 5s, page @db-team"
  common_errors:
    ECONNREFUSED: "Redis is down, check pod status"
    OOM_KILLED: "Increase memory limit in helm/values.yaml"
```

```yaml
# .agents/mcp.yml — MCP server configuration for agents
mcp:
  servers:
    gitlab:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-gitlab"]
      env:
        GITLAB_TOKEN: "${GITLAB_TOKEN}"
        GITLAB_URL: "${GITLAB_URL}"
```

---

## Org-wide distribution

The pattern scales from one repo to hundreds:

```
org/
├── .agents/              # org-level defaults (shared)
│   ├── team.yml
│   └── conventions.yml
├── service-a/
│   └── .agents/          # service overrides + additions
│       ├── agents.yml
│       └── stack.yml
└── service-b/
    └── .agents/
        ├── agents.yml
        └── deployment.yml
```

A CI job or simple shell script can merge org + repo context before injecting into an agent session. See [`docs/merging.md`](docs/merging.md).

---

## Integrations

→ [`docs/integrations.md`](docs/integrations.md) — OpenCode, Claude Code, Cursor, GitHub Copilot

---

## Philosophy

- **One convention, any tool.** The `.agents/` directory is tool-agnostic. Each integration file is a thin adapter.
- **Structured over prose.** Data agents can parse beats prose agents must interpret.
- **Minimal surface area.** No build step, no CLI, no dependencies. Just files.
- **Composable by design.** Every slice is independent. Merge as needed.
- **Agent-first.** Written for machines to consume, humans to maintain.
