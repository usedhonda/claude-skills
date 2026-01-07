---
description: Run skill evaluation against Golden Tasks
argument-hint: [--baseline|--regression CONS-XXX|--ci]
---

# learn-check

Evaluate generated skills against Golden Tasks.

## Usage

```
/skill-from-history:learn-check                        # Run all tasks
/skill-from-history:learn-check --baseline             # Compare baseline
/skill-from-history:learn-check --regression CONS-XXX  # Regression test
/skill-from-history:learn-check --ci                   # CI mode
```

## Output Example

```markdown
## Golden Task Results

| Task | Status | Duration | Notes |
|------|--------|----------|-------|
| GT-001 | PASS | 2.3s | All criteria met |
| GT-002 | FAIL | 1.8s | CONS-003 violated |

## Summary
- Passed: 1 (50%)
- Failed: 1
```

## See Also

- [learn-init](./learn-init.md) - Create Golden Tasks
- [learn-promote](./learn-promote.md) - Promote passing skills
