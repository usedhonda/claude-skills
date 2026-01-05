# parallel-dev-orchestrator

Boris流CLI⇄Web並列開発ワークフローを実現するClaude Codeスキル。

## 概要

大きなタスクを分解し、複数のClaude Codeセッションで並列開発を行い、自動マージまでを管理します。

```
/orchestrate "認証システムをOAuth2対応に"
     │
     ▼
  Plan (タスク分解)
     │
     ▼
  Dispatch (並列投入)
     │
     ▼
  Monitor (PR監視)
     │
     ▼
  Harvest (マージ・レポート)
```

## 前提条件

1. **GitHub Auto-merge有効**
   - Settings → General → Allow auto-merge ✓

2. **ブランチ保護ルール（main）**
   - Require status checks to pass ✓
   - Require pull request reviews (オプション)

3. **GitHub CLI認証済み**
   ```bash
   gh auth login
   ```

## クイックスタート

```bash
# フルワークフロー
/orchestrate "ユーザー認証をOAuth2対応にする"

# 既存プランから再開
/orchestrate --resume

# 個別コマンド
/dispatch --task T01
/harvest --watch
```

## Dispatch方式

| 方式 | 自動度 | 信頼性 |
|------|--------|--------|
| **& 方式** | 高 | 低（非公式） |
| **Worktree** | 中 | 高 |
| **手動Web** | 低 | 高 |

### Worktree方式（推奨）

```bash
# worktree作成
git worktree add .worktrees/t01 cc/YYYYMMDD-HHMM/t01-task

# 別ターミナルで
cd .worktrees/t01 && claude
```

## コマンド一覧

| コマンド | 説明 |
|---------|------|
| `/orchestrate` | フルワークフロー実行 |
| `/dispatch` | タスク投入のみ |
| `/harvest` | PR収集・レポート生成 |

## ファイル構成

```
skills/parallel-dev-orchestrator/
├── SKILL.md           # メインスキル
├── commands/          # コマンド定義
├── references/        # 詳細仕様
└── templates/         # テンプレート
```

## 実践例

### 例1: 認証機能の並列開発

```bash
# Step 1: オーケストレーション開始
/orchestrate "ユーザー認証をOAuth2 + Redis対応に"

# Step 2: plan.yamlが生成される
# reports/plan-20260105-1400.yaml
#   T01: OAuth2プロバイダー追加
#   T02: セッションRedis移行
#   T03: 認証E2Eテスト

# Step 3: worktree作成・並列作業
git worktree add .worktrees/t01 cc/20260105-1400/t01-oauth2
git worktree add .worktrees/t02 cc/20260105-1400/t02-redis

# Step 4: 別ターミナルで claude 起動
cd .worktrees/t01 && claude  # T01担当
cd .worktrees/t02 && claude  # T02担当

# Step 5: PR作成 → auto-merge待ち
/harvest --watch

# Step 6: レポート確認
cat reports/20260105-1400-report.md
```

### 例2: ドキュメント並列更新

```bash
/orchestrate "READMEとAPIドキュメントを更新"

# 生成されるタスク:
#   T01: README更新
#   T02: API仕様書更新
#   T03: 変更履歴追記

# scopeが完全分離なので安全に並列実行
```

### 例3: 既存プランから再開

```bash
# 前回の作業を再開
/orchestrate --resume

# 特定タスクのみ再投入
/dispatch --task T02

# レポートのみ生成（マージしない）
/harvest --report-only
```

## 詳細ドキュメント

- [SKILL.md](SKILL.md) - コアワークフロー
- [references/plan-format.md](references/plan-format.md) - タスクカードYAML仕様
- [references/github-setup.md](references/github-setup.md) - GitHub設定ガイド
- [references/dispatch-methods.md](references/dispatch-methods.md) - 投入方式詳細
- [references/troubleshooting.md](references/troubleshooting.md) - FAQ

## 関連

- [worktree-dispatcher](../../agents/worktree-dispatcher/) - タスク分解エージェント
