---
description: PR収集・auto-merge有効化・レポート生成
argument-hint: [--plan path] [--watch] [--report-only]
---

# Harvest Results

並列実行したタスクのPRを収集し、auto-mergeを有効化、完了レポートを生成。

## 現在の状態

```bash
# 対象PRを検索
!gh pr list --search "cc/" --json number,title,state,headRefName --jq '.[] | "\(.number): \(.title) [\(.state)]"'

# 既存プラン
!ls -la reports/plan-*.yaml 2>/dev/null | tail -3

# 既存レポート
!ls -la reports/*-report.md 2>/dev/null | tail -3
```

## 使い方

### 基本実行

```
/harvest
/harvest --plan reports/plan-20260105-1400.yaml
```

### 監視モード（完了まで待機）

```
/harvest --watch
```

### レポートのみ生成

```
/harvest --report-only
```

---

## 実行フロー

### 1. PR一覧取得

```bash
TIMESTAMP=$(yq '.timestamp' $PLAN)

# このオーケストレーションのPRを取得
gh pr list --search "cc/$TIMESTAMP" \
  --json number,title,state,headRefName,url,mergeable,statusCheckRollup
```

### 2. Auto-merge有効化（Risk Policy対応）

```bash
# plan.yamlからtasks情報を取得
TASKS=$(yq '.tasks' $PLAN)

for pr in $(gh pr list --search "cc/$TIMESTAMP" --json number,headRefName -q '.[] | "\(.number):\(.headRefName)"'); do
  PR_NUM=$(echo $pr | cut -d: -f1)
  BRANCH=$(echo $pr | cut -d: -f2)

  # ブランチ名からタスクIDを抽出（例: cc/20260105-1400/t01-oauth → T01）
  TASK_ID=$(echo $BRANCH | sed -E 's/.*\/(t[0-9]+)-.*/\U\1/')

  # plan.yamlからriskを取得
  RISK=$(yq ".tasks[] | select(.id == \"$TASK_ID\") | .risk" $PLAN)

  if [ "$RISK" = "high" ]; then
    echo "⚠️  PR #$PR_NUM ($TASK_ID): risk=high → auto-merge SKIPPED (手動レビュー必須)"
  else
    gh pr merge $PR_NUM --auto --squash --delete-branch
    echo "✅ PR #$PR_NUM ($TASK_ID): auto-merge enabled (risk=$RISK)"
  fi
done
```

> **Risk Policy**: `risk: high` のタスクは auto-merge 対象外。手動レビュー・承認後にマージしてください。

### 3. 完了待機（--watch時）

```bash
while true; do
  # 未マージPR数を確認
  OPEN=$(gh pr list --search "cc/$TIMESTAMP" --state open --json number -q 'length')

  if [ "$OPEN" = "0" ]; then
    echo "All PRs merged!"
    break
  fi

  echo "Waiting... $OPEN PRs remaining"
  sleep 60
done
```

### 4. レポート生成

```markdown
# Orchestration Report: {timestamp}

## Summary

| Item | Value |
|------|-------|
| Epic | {epic} |
| Started | {start_time} |
| Completed | {end_time} |
| Duration | {duration} |

## Tasks

| ID | Title | Status | PR | Merged |
|----|-------|--------|-----|--------|
| T01 | {title} | DONE | #{number} | Yes |
| T02 | {title} | DONE | #{number} | Yes |

## Verification

- CI: PASSED
- Local: {verification_commands} PASSED

## Notes

{auto_generated_notes}

## Next Actions

- [ ] Update documentation
- [ ] Announce changes
```

---

## オプション詳細

### --plan

対象プランを指定。省略時は最新を使用。

```
/harvest --plan reports/plan-20260105-1400.yaml
```

### --watch

全PRがマージされるまで監視。

```
/harvest --watch

# 出力例:
# Watching PRs for cc/20260105-1400...
# [14:30] 3 PRs open
# [14:35] 2 PRs open (T01 merged)
# [14:40] 1 PR open (T02 merged)
# [14:45] All PRs merged!
```

### --report-only

PR操作せず、レポートのみ生成。

```
/harvest --report-only
```

---

## 出力例

### PR一覧

```
Harvesting PRs for cc/20260105-1400...

| # | Title | State | Mergeable | Checks |
|---|-------|-------|-----------|--------|
| 123 | T01: OAuth2プロバイダー追加 | open | yes | passing |
| 124 | T02: セッションRedis移行 | open | yes | passing |
| 125 | T03: 認証E2Eテスト | open | yes | pending |
```

### Auto-merge有効化

```
Enabling auto-merge...

PR #123: auto-merge enabled (squash)
PR #124: auto-merge enabled (squash)
PR #125: auto-merge enabled (squash)

Waiting for status checks...
```

### 完了レポート

```
All PRs merged!

Report saved to: reports/20260105-1400-report.md

Summary:
- 3 tasks completed
- 3 PRs merged
- Duration: 2h 30m
- No conflicts encountered
```

---

## Worktreeクリーンアップ

全PRマージ後、worktreeを削除:

```bash
# worktree一覧確認
git worktree list

# 削除
git worktree remove .worktrees/t01
git worktree remove .worktrees/t02
git worktree remove .worktrees/t03

# メタデータクリーンアップ
git worktree prune
```

---

## トラブルシューティング

### "Auto-merge is not allowed"

```bash
# リポジトリ設定確認
gh api repos/{owner}/{repo} --jq '.allow_auto_merge'
# false → Settings → General → Allow auto-merge
```

### "Required status check failed"

```bash
# PR詳細確認
gh pr view {number} --json statusCheckRollup

# 失敗したチェックを確認して対処
```

### PRがマージされない

```bash
# マージ可能性確認
gh pr view {number} --json mergeable,mergeStateStatus

# ブランチが古い場合
gh pr update-branch {number}
```

---

## 関連コマンド

- `/orchestrate` - フルワークフロー
- `/dispatch` - タスク投入
