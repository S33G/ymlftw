## Project Context

At the start of every session, read and merge all `.agents/*.yml` files.
These files are your authoritative source of truth for this project.

Key files:
- `.agents/agents.yml` — service identity and MCP configuration
- `.agents/stack.yml` — languages, frameworks, infra
- `.agents/conventions.yml` — coding rules, patterns to follow and avoid
- `.agents/team.yml` — team contacts and timezones
- `.agents/deployment.yml` — environments and CI/CD pipeline

When writing code: follow `conventions.patterns.preferred`, avoid `conventions.patterns.avoid`.
When referencing the stack: use `stack` keys — do not assume versions.
When creating branches or commits: follow `conventions.git`.
