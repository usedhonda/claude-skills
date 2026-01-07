---
description: Dispatch planned tasks for parallel execution
argument-hint: [--task T01|--method worktree|web]
---

# par-dispatch

Submit tasks from active plan for parallel execution.

## Usage

```
/parallel-dev-orchestrator:par-dispatch                     # All tasks
/parallel-dev-orchestrator:par-dispatch --task T01          # Specific task
/parallel-dev-orchestrator:par-dispatch --method worktree   # Worktree method
```

## Methods

| Method | Command | Description |
|--------|---------|-------------|
| & | `--method &` | Boris-style (default) |
| Worktree | `--method worktree` | Create git worktrees |
| Web | `--method web` | Instructions only |

## Worktree Method

```
Run in separate terminal:
  cd .worktrees/t01 && claude
```

## See Also

- [par-plan](./par-plan.md) - Create plan first
- [par-status](./par-status.md) - Monitor progress
- [par-harvest](./par-harvest.md) - Collect completed PRs
