---
description: Create parallel task plan from epic
argument-hint: "epic description"
---

# par-plan

Decompose an epic into parallel tasks with isolated scopes.

## Usage

```
/parallel-dev-orchestrator:par-plan "Add OAuth2 authentication"
```

## Process

1. Analyze epic
2. Decompose into 2-5 tasks
3. Define scope (include/exclude) for each
4. Save to `reports/plan-{timestamp}.yaml`

## Output Format

```yaml
epic: "Add OAuth2 authentication"
base_branch: "main"
timestamp: "20260106-1500"

tasks:
  - id: T01
    title: "OAuth2 provider"
    branch: "cc/20260106-1500/t01-oauth2"
    scope:
      include: ["src/auth/providers/"]
      exclude: ["src/auth/session/"]
    done:
      - "Google login works with OAuth2"
      - "npm test -- auth/providers passes"
    risk: medium
    dependencies: []

merge_order: ["T01", "T02"]

verification:
  pre_merge: ["npm test", "npm run lint"]
```

## Scope Rules

| Rule | Description |
|------|-------------|
| Scope isolation | No file overlap between tasks |
| Exclude required | Other tasks' includes go to exclude |
| Verifiable done | Include test commands |
| Risk level | `high` = no auto-merge |

## See Also

- [par-dispatch](./par-dispatch.md) - Submit planned tasks
