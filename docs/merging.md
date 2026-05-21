# Merging Context

How to merge org-level and repo-level `.agents/` files.

---

## The concept

Each repo has its own `.agents/`. An org can also maintain shared `.agents/` defaults. Deep-merge them — repo wins on conflict.

```
org-defaults/.agents/team.yml        → base team info
my-service/.agents/team.yml          → override specific members
─────────────────────────────────────
merged result: org defaults + service overrides
```

---

## Shell (yq)

Requires [`yq`](https://github.com/mikefarah/yq).

```bash
# Merge all .agents/*.yml files into one
yq eval-all '. as $item ireduce ({}; . * $item)' .agents/*.yml
```

```bash
# Merge org defaults + repo context
yq eval-all '. as $item ireduce ({}; . * $item)' \
  /path/to/org-defaults/.agents/*.yml \
  .agents/*.yml
```

```bash
# Output as JSON (for jq pipelines)
yq eval-all -o=json '. as $item ireduce ({}; . * $item)' .agents/*.yml
```

---

## Node.js

```js
import { readFileSync, readdirSync } from 'fs'
import { join } from 'path'
import { load } from 'js-yaml'

function mergeAgentContext(dirs = ['.agents']) {
  return dirs
    .flatMap(dir =>
      readdirSync(dir)
        .filter(f => f.endsWith('.yml') || f.endsWith('.yaml'))
        .map(f => load(readFileSync(join(dir, f), 'utf8')))
    )
    .reduce((acc, obj) => deepMerge(acc, obj), {})
}

function deepMerge(target, source) {
  const result = { ...target }
  for (const key of Object.keys(source ?? {})) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      result[key] = deepMerge(target[key] ?? {}, source[key])
    } else {
      result[key] = source[key]
    }
  }
  return result
}
```

---

## Python

```python
import yaml
import glob
from functools import reduce

def deep_merge(base, override):
    result = dict(base)
    for k, v in (override or {}).items():
        if isinstance(v, dict) and isinstance(result.get(k), dict):
            result[k] = deep_merge(result[k], v)
        else:
            result[k] = v
    return result

def load_agent_context(*dirs):
    files = [f for d in dirs for f in sorted(glob.glob(f"{d}/*.yml"))]
    return reduce(deep_merge, [yaml.safe_load(open(f)) or {} for f in files], {})

ctx = load_agent_context(".agents")
```

---

## CI/CD (GitLab)

Inject merged context as an artifact or env var:

```yaml
# .gitlab-ci.yml
generate-context:
  stage: .pre
  script:
    - yq eval-all -o=json '. as $item ireduce ({}; . * $item)' .agents/*.yml > .agent-context.json
  artifacts:
    paths:
      - .agent-context.json
```

---

## Array merging

By default, deep-merge **replaces** arrays. To **append** instead, use `yq`'s `+` merge strategy:

```bash
yq eval-all '. as $item ireduce ({}; . *+ $item)' .agents/*.yml
```

The `*+` operator concatenates arrays rather than replacing them — useful for `team.members`, `conventions.patterns.preferred`, etc.
