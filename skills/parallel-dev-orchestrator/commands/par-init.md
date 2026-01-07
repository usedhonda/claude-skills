---
description: Check parallel development environment
---

# par-init

Verify environment requirements for parallel development.

## Usage

```
/parallel-dev-orchestrator:par-init
```

## Checks

```
Environment Check

GitHub CLI:
  gh: installed (2.40.0)
  authenticated: yes

Repository:
  allow_auto_merge: enabled
  branch_protection: main protected
  required_checks: CI configured

Tools:
  yq: installed
  jq: installed

Result: Ready for parallel development
```

## Fix: auto_merge disabled

```
1. Settings -> General -> Pull Requests
2. Enable "Allow auto-merge"

Or:
  gh api repos/{owner}/{repo} -X PATCH -f allow_auto_merge=true
```

## See Also

- [par-plan](./par-plan.md) - Create task plan after init
