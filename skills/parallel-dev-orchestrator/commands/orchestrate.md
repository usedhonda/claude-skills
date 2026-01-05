---
description: Boris流CLI⇄Web並列開発のフルワークフロー実行
argument-hint: [epic説明] or [--resume] or [--from-plan]
---

# Orchestrate Parallel Development

Plan → Dispatch → Monitor → Harvest の全フェーズを実行。

## 現在の状態

```bash
# Git status
!git status --porcelain

# Current branch
!git branch --show-current

# Remote URL
!git remote get-url origin 2>/dev/null || echo "No remote"
```

## 使い方

### 新規実行

```
/orchestrate "ユーザー認証をOAuth2対応にする"
```

### 既存プランから再開

```
/orchestrate --resume
```

### plan-parallel から実行

```
/orchestrate --from-plan
```

`/plan-parallel` で作成した計画ファイル（`.claude/plans/*.md`）から
YAMLブロックを抽出し、`reports/plan-{timestamp}.yaml` に保存して実行。

---

## 実行フロー

### Phase 1: Plan

#### 通常実行の場合

**worktree-dispatcher** エージェントを使ってタスク分解:

1. epicを分析
2. 2-5個のタスクに分解
3. 各タスクのscope（include/exclude）を決定
4. `reports/plan-{timestamp}.yaml` に保存

#### --from-plan の場合

`/plan-parallel` で作成済みの計画を使用:

```bash
# 最新の計画ファイルを取得
PLAN_FILE=$(ls -t .claude/plans/*.md | head -1)

# YAMLブロックを抽出
YAML=$(sed -n '/^```yaml/,/^```/p' "$PLAN_FILE" | sed '1d;$d')

# reports/plan-{timestamp}.yaml に保存
TIMESTAMP=$(date '+%Y%m%d-%H%M')
echo "$YAML" > "reports/plan-$TIMESTAMP.yaml"
```

→ Phase 2 (Branch Setup) へスキップ

**出力形式**:

```yaml
epic: "{ユーザーのepic説明}"
base_branch: "main"
timestamp: "{YYYYMMDD-HHMM}"
tasks:
  - id: T01
    title: "タスク名"
    branch: "cc/{timestamp}/t01-slug"
    scope:
      include: ["触るパス"]
      exclude: ["触らないパス"]
    done:
      - "完了条件"
      - "テストコマンド"
    risk: low|medium|high
    dependencies: []
merge_order: ["T01", "T02"]
verification:
  pre_merge: ["npm test"]
```

### Phase 2: Branch Setup

各タスク用ブランチを作成:

```bash
git checkout main
git pull --rebase

# 各タスクについて
git checkout -b cc/{timestamp}/{task-id}-{slug}
git commit --allow-empty -m "chore: start {task-id}"
git push -u origin cc/{timestamp}/{task-id}-{slug}
git checkout main
```

### Phase 3: Dispatch

**優先順位**:

1. **& 方式（Boris式）** - 末尾に `&` を付けてWebに投入
2. **Worktree方式** - 失敗時のフォールバック
3. **手動Web** - 最終手段

**& 方式の実行**:

```
{task-prompt-from-template} &
```

**Worktree方式の実行**:

```bash
# worktree作成
git worktree add .worktrees/{task-id} cc/{timestamp}/{task-id}-{slug}

# ユーザーに通知
echo "別ターミナルで以下を実行:"
echo "cd .worktrees/{task-id} && claude"
```

### Phase 4: Monitor & Auto-merge

```bash
# PR一覧取得
gh pr list --search "cc/{timestamp}" --json number,title,state,url

# Risk Policyに従ってauto-merge（/harvest に委譲）
/harvest --plan reports/plan-{timestamp}.yaml
```

> **Risk Policy**: `risk: high` タスクは auto-merge 対象外（手動レビュー必須）

### Phase 5: Harvest

全PRマージ完了後:

1. 完了レポート生成 → `reports/{timestamp}-report.md`
2. worktreeクリーンアップ
3. ブランチ削除確認

---

## 重要な制約

1. **ブランチ保護必須** - auto-merge動作の前提
2. **scope分離必須** - 並列作業のコンフリクト防止
3. **1タスク = 1ブランチ = 1PR** - 追跡可能性

---

## トラブルシューティング

### & が動作しない

→ Worktree方式にフォールバック:

```bash
git worktree add .worktrees/t01 {branch}
cd .worktrees/t01
claude
```

### auto-merge有効化失敗

→ GitHub設定確認:

```bash
gh api repos/{owner}/{repo} --jq '.allow_auto_merge'
# false なら Settings → General → Allow auto-merge
```

### コンフリクト発生

→ scope.exclude設定を確認、重複があれば分離

---

## 関連コマンド

- `/plan-parallel` - 並列開発前提の計画モード
- `/dispatch` - タスク投入のみ
- `/harvest` - PR収集・レポートのみ
