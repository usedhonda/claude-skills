---
description: Show parallel development PR status
---

# par-status

Display current parallel development status including active plans and PRs.

## Usage

```
/parallel-dev-orchestrator:par-status
```

## Output Example

```
Parallel Development Status

Active Plan: reports/plan-20260106-1500.yaml
  Epic: Add OAuth2 authentication
  Timestamp: 20260106-1500
  Tasks: 3

PRs:
  #123 T01: OAuth2 provider [OPEN] checks passed
  #124 T02: Session Redis migration [OPEN] checks passed
  #125 T03: Auth E2E tests [OPEN] checks pending

Blockers:
  T03: status checks pending

Next Action:
  Wait for checks, then /parallel-dev-orchestrator:par-harvest
```

## See Also

- [par-harvest](./par-harvest.md) - Collect and merge PRs
