---
description: Create worktrees for parallel CLI execution
argument-hint: [--task T01]
---

# par-dispatch

Worktree方式で並列CLIセッションを作成。

**Note**: Web実行は `/par-plan` で完結します（ブランチ作成・プロンプト生成まで自動）。

## Usage

```
/parallel-dev-orchestrator:par-dispatch              # 全タスク
/parallel-dev-orchestrator:par-dispatch --task T01   # 特定タスク
```

## Process

1. アクティブな計画を `reports/plan-*.yaml` から読み込み
2. 各タスク用のWorktreeを作成
3. タスクプロンプトを `.claude-task.md` として保存

**Claudeが自動実行:**

```bash
git worktree add .worktrees/t01 cc/{timestamp}/t01-xxx
git worktree add .worktrees/t02 cc/{timestamp}/t02-xxx
```

## Output

```
Worktrees created:
  .worktrees/t01/ (branch: cc/20260107-1500/t01-readme)
  .worktrees/t02/ (branch: cc/20260107-1500/t02-docs)

別ターミナルで実行:
  cd .worktrees/t01 && claude
  cd .worktrees/t02 && claude

タスク内容は各ディレクトリの .claude-task.md を参照
```

## When to Use

| 状況 | 推奨 |
|------|------|
| Webで並列実行したい | `/par-plan` で完結 |
| CLIで並列実行したい | `/par-dispatch` でWorktree作成 |
| 特定タスクだけ再実行 | `/par-dispatch --task T01` |

## See Also

- [par-plan](./par-plan.md) - 計画作成（Web実行はこちらで完結）
- [par-status](./par-status.md) - 進捗確認
- [par-harvest](./par-harvest.md) - PRをマージ
