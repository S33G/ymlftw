## Agent Context

At the start of every session, read and deep-merge all `.agents/*.yml` files.
Treat the merged object as your authoritative project context.

Key files:
- `.agents/agents.yml` — service identity, linked services, MCP config
- `.agents/stack.yml` — languages, frameworks, databases, infra
- `.agents/conventions.yml` — coding rules, naming, patterns to follow/avoid
- `.agents/team.yml` — team members, contacts, timezones
- `.agents/deployment.yml` — environments, pipeline stages, secrets
- `.agents/gitlab.yml` — branch strategy, merge workflow

When writing code: follow `conventions.patterns.preferred`, avoid `conventions.patterns.avoid`.
When referencing infrastructure: use `stack` keys — do not assume versions.
When creating branches or commits: follow `conventions.git`.
