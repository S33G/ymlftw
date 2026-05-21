# Schema Reference

Suggested key structure for `.agents/*.yml` files. Nothing is enforced — use what's relevant, add anything extra.

---

## Top-level keys

| Key | File | Purpose |
|---|---|---|
| `agent` | `agents.yml` | Service identity, URLs, MCP config |
| `team` | `team.yml` | People, roles, contacts |
| `stack` | `stack.yml` | Technologies and versions |
| `conventions` | `conventions.yml` | Coding rules and patterns |
| `deployment` | `deployment.yml` | Environments, CI/CD |
| `gitlab` | `gitlab.yml` | VCS workflow config |
| `context` | `context.yml` | Domain knowledge, glossary |
| `runbooks` | `runbooks.yml` | Operational guides |
| `mcp` | any | MCP server definitions |

Keys can live in any file. The top-level key is what matters after merging.

---

## `agent`

```yaml
agent:
  name: string
  description: string
  url: string
  branch: string
  docs: string
  dashboard: string
  status: string
  integrations:
    - name: string
      url: string
      relationship: upstream | downstream | peer
  mcp:
    <server-name>:
      command: string
      args: string[]
      env:
        <KEY>: string
```

---

## `team`

```yaml
team:
  name: string
  members:
    - name: string
      role: string
      contact: string
      timezone: string
      slack: string | null
      github: string | null
      gitlab: string | null
  contact:
    email: string
    slack: string
    timezone: string
```

---

## `stack`

```yaml
stack:
  runtime: string          # e.g. "node@22"
  language: string         # e.g. "typescript@5.4"
  package_manager: string
  frameworks: string[]
  databases:
    primary: string
    cache: string
    search: string
  infrastructure:
    cloud: aws | gcp | azure
    container: docker | podman
    orchestration: kubernetes | nomad | ecs
    iac: terraform | pulumi | cdk
  testing:
    unit: string
    e2e: string
    coverage_threshold: number
  ci:
    platform: string
    registry: string
```

---

## `conventions`

```yaml
conventions:
  style:
    formatter: string
    linter: string
    config: string
  patterns:
    preferred: string[]
    avoid: string[]
  naming:
    files: kebab-case | snake_case | camelCase
    types_interfaces: string
    functions_variables: string
    env_vars: string
    database_tables: string
  git:
    branch_prefix: string
    commit_style: conventional-commits | gitmoji | free
    pr_template: string
```

---

## `deployment`

```yaml
deployment:
  environments:
    <name>:
      url: string
      branch: string
      auto_deploy: boolean
      approval_required: boolean
  pipeline:
    stages: string[]
    test_command: string
    build_command: string
    docker_file: string
  secrets:
    manager: string
    required: string[]
```

---

## `context` (free-form domain knowledge)

```yaml
context:
  domain: string
  glossary:
    <term>: string
  invariants: string[]
  external_services:
    - name: string
      url: string
      notes: string
```

---

## `runbooks`

```yaml
runbooks:
  on_call: string[]
  common_errors:
    <error_pattern>: string
  escalation:
    p1: string
    p2: string
```

---

## Adding your own keys

Just add them. There's no registry. Document what you add here for your team.
