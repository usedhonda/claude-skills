---
description: Harvest and merge completed PRs
argument-hint: [--watch|--report-only]
---

# par-harvest

Collect completed PRs and enable auto-merge based on risk level.

## Usage

```
/parallel-dev-orchestrator:par-harvest                      # Harvest + auto-merge
/parallel-dev-orchestrator:par-harvest --watch              # Wait for completion
/parallel-dev-orchestrator:par-harvest --report-only        # Report only
```

## Risk Policy

| Risk | Auto-merge |
|------|------------|
| low | Enabled |
| medium | Enabled |
| high | Manual review required |

## Completion Report

```markdown
# Orchestration Report: 20260106-1500

## Summary
| Item | Value |
|------|-------|
| Epic | Add OAuth2 authentication |
| Duration | 2h 30m |

## Tasks
| ID | Title | Status | PR |
|----|-------|--------|-----|
| T01 | OAuth2 provider | DONE | #123 |
| T02 | Session Redis | DONE | #124 |
```

## See Also

- [par-status](./par-status.md) - Check status before harvest
