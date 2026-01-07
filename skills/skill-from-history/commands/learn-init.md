---
description: Initialize Golden Tasks for skill evaluation
---

# learn-init

Create Golden Tasks scaffold for evaluating generated skills.

## Usage

```
/skill-from-history:learn-init
```

## Created Files

```
.claude/golden-tasks/
  index.yaml          # Task index
  GT-001.yaml         # Sample task
  README.md           # Creation guide
```

## Golden Task Format

```yaml
id: GT-001
name: "Basic code generation"
input:
  context: "Express.js project"
  request: "Add user auth endpoint"
expected:
  constraints_applied: ["CONS-001", "CONS-002"]
  patterns_used: ["error-handling", "validation"]
  violations: []
```

## See Also

- [learn-check](./learn-check.md) - Run evaluation against Golden Tasks
