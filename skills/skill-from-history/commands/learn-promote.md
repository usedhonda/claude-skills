---
description: Promote skill maturity level
argument-hint: [CONS-XXX|--all-eligible|--diff CONS-XXX]
---

# learn-promote

Promote skills through maturity levels based on evaluation results.

## Usage

```
/skill-from-history:learn-promote                    # Show candidates
/skill-from-history:learn-promote CONS-XXX           # Promote specific
/skill-from-history:learn-promote --all-eligible     # Promote all eligible
/skill-from-history:learn-promote --diff CONS-XXX    # Preview diff
```

## Maturity Levels

| Level | Enforcement | Requirements |
|-------|-------------|--------------|
| Draft | Warning | Initial state |
| Accepted | Block (bypassable) | Eval >= 80% |
| Canonical | Block (fixed) | Eval >= 95%, 30 days stable |

## See Also

- [learn-check](./learn-check.md) - Run evaluation first
- [learn-status](./learn-status.md) - View current maturity
