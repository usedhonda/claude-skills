# Troubleshooting Guide

並列開発オーケストレーションでよくある問題と解決策。

---

## Dispatch Issues

### `&` が動作しない

**症状**: プロンプト末尾に`&`を付けてもWebセッションが開始されない

**原因**: `&`は公式ドキュメント未記載の機能

**解決策**:

```bash
# Worktree方式にフォールバック
git worktree add .worktrees/t01 cc/20260105-1400/t01-oauth2

# 別ターミナルで
cd .worktrees/t01
claude
# タスクプロンプトを貼り付け
```

---

### Worktreeが作成できない

**症状**: `git worktree add` が失敗

**原因A**: ブランチが存在しない

```bash
# 確認
git branch -a | grep cc/20260105

# 解決: ブランチを作成・push
git checkout -b cc/20260105-1400/t01-oauth2
git push -u origin cc/20260105-1400/t01-oauth2
git checkout main
```

**原因B**: 同じブランチが既にworktreeで使用中

```bash
# 確認
git worktree list

# 解決: 既存worktreeを削除
git worktree remove .worktrees/t01
```

**原因C**: ディレクトリが既に存在

```bash
# 確認
ls -la .worktrees/

# 解決: 手動削除
rm -rf .worktrees/t01
git worktree prune
```

---

### ブランチ名の不一致

**症状**: WebセッションとCLIでブランチが違う

**原因**: ブランチを事前pushしていない

**解決策**:

```bash
# 空コミットでブランチを固定
git checkout main
git checkout -b cc/20260105-1400/t01-oauth2
git commit --allow-empty -m "chore: start T01"
git push -u origin cc/20260105-1400/t01-oauth2
git checkout main
```

---

## Auto-merge Issues

### "Auto-merge is not allowed"

**症状**: `gh pr merge --auto` が失敗

**原因**: リポジトリ設定でauto-merge無効

**解決策**:

1. GitHub → Repository → Settings
2. General → Pull Requests
3. **Allow auto-merge** にチェック

```bash
# 確認
gh api repos/{owner}/{repo} --jq '.allow_auto_merge'
```

---

### "Required status check is expected"

**症状**: auto-mergeが有効にならない

**原因**: CIが未設定または名前不一致

**解決策**:

1. CIを一度実行させる（PRを作成）
2. Settings → Branches → main → Edit
3. "Require status checks"で正しい名前を選択

```bash
# 利用可能なチェック名を確認
gh api repos/{owner}/{repo}/branches/main/protection/required_status_checks \
  --jq '.contexts[]'
```

---

### "Review required"

**症状**: レビューがないためマージできない

**原因**: Branch protectionでレビュー必須設定

**解決策A（ソロ開発）**:

Settings → Branches → main → Edit
- "Required approving reviews" を 0 に

**解決策B（チーム開発）**:

```bash
# 自分でApprove
gh pr review {number} --approve
```

---

### PRがマージされない（pending状態）

**症状**: auto-merge有効だがマージされない

**原因A**: ステータスチェックが未完了

```bash
# 確認
gh pr view {number} --json statusCheckRollup --jq '.statusCheckRollup'
```

**原因B**: ブランチが古い

```bash
# 最新に更新
gh pr update-branch {number}
```

**原因C**: コンフリクト

```bash
# 確認
gh pr view {number} --json mergeable

# 解決: 手動でrebase
git checkout cc/20260105-1400/t01-oauth2
git rebase main
git push --force-with-lease
```

---

## Conflict Issues

### コンフリクトが発生

**症状**: PRマージ時にコンフリクト

**原因**: scope分離が不十分

**解決策（短期）**:

```bash
# 手動で解決
git checkout cc/20260105-1400/t01-oauth2
git rebase main
# コンフリクト解決
git add .
git rebase --continue
git push --force-with-lease
```

**解決策（長期）**:

plan.yamlのscope.excludeを見直し:

```yaml
tasks:
  - id: T01
    scope:
      include: ["src/auth/providers/"]
      exclude:
        - "src/auth/session/"  # T02の領域
        - "src/auth/middleware/"  # 追加

  - id: T02
    scope:
      include: ["src/auth/session/"]
      exclude:
        - "src/auth/providers/"  # T01の領域
```

---

### マージ順序の問題

**症状**: 依存関係のあるPRが先にマージされようとする

**原因**: merge_orderと実際のマージ順序が不一致

**解決策**:

```bash
# 依存先がマージされるまで待つ
gh pr merge {T03_number} --auto --squash

# または手動でマージ順序を制御
gh pr merge {T01_number} --squash --delete-branch
# T01マージ完了を確認
gh pr merge {T02_number} --squash --delete-branch
# T02マージ完了を確認
gh pr merge {T03_number} --squash --delete-branch
```

---

## Session Issues

### Webセッションが見つからない

**症状**: `&`で投入したが進捗確認できない

**解決策**:

1. claude.ai/code を開く
2. 対象リポジトリを選択
3. 最近のセッションを確認

または:

```bash
# PRが作成されているか確認
gh pr list --search "cc/20260105"
```

---

### Teleportが失敗

**症状**: Web→CLIのteleportがエラー

**原因A**: ローカルに未コミット変更がある

```bash
# 確認
git status

# 解決: stash
git stash
claude --teleport {session-id}
git stash pop
```

**原因B**: セッションIDが無効

```bash
# 正しいセッションIDを取得
# claude.ai/code → "Open in CLI" → コマンドをコピー
```

---

## Performance Issues

### Worktreeが多すぎる

**症状**: ディスク容量不足

**解決策**:

```bash
# 不要なworktreeを確認
git worktree list

# 削除
git worktree remove .worktrees/t01
git worktree remove .worktrees/t02
git worktree prune

# 強制削除（未コミット変更がある場合）
git worktree remove -f .worktrees/t01
```

---

### PRが大量に残っている

**症状**: 古いPRがマージされずに残っている

**解決策**:

```bash
# 古いPRをクローズ
gh pr list --state open --json number,title --jq '.[] | select(.title | startswith("cc/")) | .number' | \
  xargs -I {} gh pr close {}
```

---

## Quick Reference

| 症状 | 確認コマンド | 解決策 |
|------|-------------|--------|
| & 動作しない | - | Worktree方式へ |
| auto-merge無効 | `gh api repos/.../allow_auto_merge` | Settings有効化 |
| チェック失敗 | `gh pr view --json statusCheckRollup` | CI修正 |
| コンフリクト | `gh pr view --json mergeable` | rebase |
| ブランチ古い | `gh pr view --json mergeStateStatus` | update-branch |
| worktree残骸 | `git worktree list` | remove + prune |
