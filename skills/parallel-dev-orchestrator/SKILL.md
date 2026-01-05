---
name: parallel-dev-orchestrator
description: |
  Boris流CLI⇄Web並列開発ワークフロー。
  タスク分解→ブランチ作成→並列投入→auto-merge→レポート。
  "/orchestrate", "並列開発", "タスク分解", "parallel development"で発動。
compression-anchors:
  - "& でWeb投入、失敗時はworktreeフォールバック"
  - "Plan→Dispatch→Monitor→Harvest"
  - "1タスク = 1ブランチ = 1PR"
agents:
  - worktree-dispatcher
---

# Parallel Dev Orchestrator

CLI⇄Web並列開発を自動化するBoris流ワークフロー。

## TL;DR

1. **Plan**: epicをタスク分解 → `reports/plan-{timestamp}.yaml`
2. **Dispatch**: 各タスクをWeb or worktreeに投入
3. **Harvest**: PR監視 → auto-merge → レポート生成

---

## When to Use

- 複数の独立した作業を並列で進めたい
- 1つの大きな機能を分割して開発したい
- Claude Code Webも活用して開発速度を上げたい

**Trigger phrases**: `/orchestrate`, `並列開発`, `タスク分解`, `parallel tasks`

---

## When NOT to Use

このスキルは強力だが、以下のケースでは**使わない方が良い**：

| ケース | 理由 | 代替案 |
|--------|------|--------|
| 大規模リネーム/リファクタ | 複数ブランチで同一ファイルを触る | 単一ブランチで実行 |
| 広域の設計変更 | scope分離が困難 | 先に設計PRを1つ作成 |
| DBマイグレーション | 実行順序が厳密 | 依存順に直列実行 |
| 同一ファイル集合を触るタスク | コンフリクト確定 | タスクを再分割 |
| 依存が鎖状に長いepic | 並列化の利点がない | 依存順に直列実行 |
| 初めてのリポジトリ | 構造理解が先 | 単一タスクで試行 |

**判断基準**: scope.excludeで明確に分離できないなら、並列化しない。

---

## Risk Policy

タスクの `risk` レベルに応じた運用ルール：

| Risk | auto-merge | 必須条件 | 推奨アクション |
|------|------------|----------|----------------|
| **low** | ✅ 許可 | CI通過 | `--watch` で監視 |
| **medium** | ✅ 許可 | CI通過 + required checks | PR内容を軽く確認 |
| **high** | ❌ 禁止 | 手動レビュー必須 | `/harvest --report-only` で状況確認後、手動マージ |

**High riskの例**: 認証/決済/データ削除/外部API連携/本番設定変更

**運用ルール詳細**:
- **medium**: 差分の目視確認 + 主要テスト通過 + 影響範囲がscope内であること
- **high**: 最低1 approval必須、squashマージ固定、`/harvest`は`--auto`を実行しない

```yaml
# plan.yaml でのrisk指定
tasks:
  - id: T01
    risk: high  # → auto-merge対象外、手動レビュー必須
```

> **実装**: `risk: high` のタスクに対して `/harvest` は `gh pr merge --auto` を実行しない

---

## Prerequisites

### 1. GitHub Auto-merge有効化

```
Repo Settings → General → Allow auto-merge ✓
```

### 2. ブランチ保護ルール（main）

```
Settings → Branches → Add branch protection rule

必須設定:
- Require status checks to pass before merging ✓
- Require pull request reviews before merging ✓ (オプション)
- Require branches to be up to date before merging ✓
```

> **重要**: これがないと`gh pr merge --auto`が機能しない

### 3. GitHub CLI認証

```bash
gh auth login
gh auth status  # 確認
```

### 4. 必須ツール

```bash
# yq (YAML処理)
brew install yq  # macOS
# または: pip install yq

# jq (JSON処理)
brew install jq  # macOS
```

詳細: [references/github-setup.md](references/github-setup.md)

---

## Core Workflow

```
┌─────────────────────────────────────────────────────────┐
│  /orchestrate "認証機能をOAuth2対応に"                    │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 1: Plan (worktree-dispatcher)                    │
│  ─────────────────────────────────────                  │
│  epicを分析 → タスクカード生成                            │
│  → reports/plan-20260105-1400.yaml                      │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 2: Branch Setup                                  │
│  ─────────────────────────────────────                  │
│  各タスク用ブランチを作成・push                           │
│  cc/20260105-1400/t01-oauth2-provider                   │
│  cc/20260105-1400/t02-session-redis                     │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 3: Dispatch                                      │
│  ─────────────────────────────────────                  │
│  [優先] & でWebに投入                                    │
│  [代替] worktreeで並列CLI起動                            │
│  [手動] claude.ai/code に貼り付け                        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 4: Monitor & Auto-merge                          │
│  ─────────────────────────────────────                  │
│  gh pr list でPR監視                                    │
│  gh pr merge --auto でマージ有効化                       │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 5: Harvest                                       │
│  ─────────────────────────────────────                  │
│  全PRマージ完了を検知                                    │
│  → reports/20260105-1400-report.md                      │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Start

### 1. フルワークフロー

```
/orchestrate "ユーザー認証をOAuth2対応にする"
```

### 2. 既存プランから再開

```
/orchestrate --resume
```

### 3. 個別コマンド

```bash
/dispatch --task T01          # 特定タスクのみ投入
/harvest --watch              # 監視モードでマージ待ち
```

---

## Dispatch Methods

### 方式1: & でWeb投入（Boris式・推奨）

```
T01: OAuth2プロバイダー追加

Branch: cc/20260105-1400/t01-oauth2
Base: main

Scope:
- include: src/auth/, config/oauth.ts
- exclude: src/auth/legacy/

Done:
- OAuth2ログインがGoogle対応
- テスト通過: npm test -- auth

Rules:
- excludeパスは触らない
- PR作成時: "T01: OAuth2プロバイダー追加"

Start now. &
```

> **注意**: `&` は公式ドキュメント未記載。動作しない場合は方式2へ

### 方式2: Git Worktree（確実）

```bash
# worktree作成
git worktree add .worktrees/t01 cc/20260105-1400/t01-oauth2

# 別ターミナルで
cd .worktrees/t01
claude  # タスクプロンプトを貼り付け
```

### 方式3: 手動Web投入

1. claude.ai/code を開く
2. リポジトリを選択
3. タスクプロンプトを貼り付け
4. ブランチ指定して実行

詳細: [references/dispatch-methods.md](references/dispatch-methods.md)

---

## Task Card Format

```yaml
epic: "機能説明"
base_branch: "main"
timestamp: "20260105-1400"
tasks:
  - id: T01
    title: "OAuth2プロバイダー追加"
    branch: "cc/20260105-1400/t01-oauth2"
    scope:
      include: ["src/auth/", "config/oauth.ts"]
      exclude: ["src/auth/legacy/"]
    done:
      - "OAuth2ログインがGoogle対応"
      - "npm test -- auth 通過"
    risk: medium
    dependencies: []
  - id: T02
    title: "セッションRedis移行"
    branch: "cc/20260105-1400/t02-redis"
    scope:
      include: ["src/session/"]
      exclude: ["src/auth/"]
    done:
      - "再起動後もセッション維持"
      - "npm test -- session 通過"
    risk: low
    dependencies: []
merge_order: ["T01", "T02"]
verification:
  pre_merge: ["npm test", "npm run lint"]
```

**重要原則**:
- **1タスク = 1ブランチ = 1PR**
- **scope.exclude**: 同時編集を防ぐ（コンフリクト回避）
- **merge_order**: 依存関係がある場合の順序

詳細: [references/plan-format.md](references/plan-format.md)

---

## Constraints

| パターン | 重大度 | 理由 |
|---------|--------|------|
| ブランチ保護なしでauto-merge | Critical | マージ事故の原因 [E1] |
| scope重複のまま並列実行 | Critical | コンフリクト確定 [E2] |
| 大きすぎるタスク（>500行） | Warning | レビュー困難 [E3] |
| テストなしでマージ | Warning | 品質低下 [E3] |

---

## Recovery Playbook

問題発生時の標準手順：

### 1. Dispatch失敗

```bash
# 状態確認
git worktree list
gh pr list --state open

# リカバリ
git worktree remove .worktrees/t01  # 問題のworktree削除
git branch -D cc/xxx/t01-xxx        # ブランチ削除
# → 再度 /dispatch --task T01
```

### 2. PRがコンフリクト

```bash
# mainを取り込んで解決
cd .worktrees/t01
git fetch origin main
git rebase origin/main
# コンフリクト解決後
git push --force-with-lease
```

### 3. CIが落ちる

```bash
# ログ確認
gh pr checks <PR番号>

# 修正してpush
cd .worktrees/t01
# 修正作業...
git add . && git commit -m "fix: CI error"
git push
```

### 4. scope逸脱（exclude触った）

```bash
# 差分確認
cd .worktrees/t01
git diff --name-only origin/main

# 該当ファイルをrevert
git checkout origin/main -- path/to/excluded/file
git commit -m "revert: remove out-of-scope changes"
git push
```

### 5. 中断→再開

```bash
# 現状確認
/harvest --report-only

# 完了タスクを確認し、残りを再投入
/dispatch --task T03 --task T04
```

### 6. リモート後始末（完全クリーンアップ）

```bash
# リモートブランチ削除
git push origin --delete cc/xxx/t01-xxx

# PRクローズ（必要時）
gh pr close <PR番号>

# ローカル掃除（最後に必ず実行）
git worktree prune
git branch -D cc/xxx/t01-xxx  # ローカルブランチ削除
```

---

## Artifacts

このスキルが生成するファイル：

| ファイル | 説明 | 生成タイミング |
|----------|------|----------------|
| `reports/plan-{timestamp}.yaml` | タスク分解結果 | /orchestrate Phase 1 |
| `.worktrees/t{N}/` | 並列作業ディレクトリ | /dispatch (worktree方式) |
| `reports/{timestamp}-report.md` | 完了レポート | /harvest |

---

## Commands

| コマンド | 説明 |
|---------|------|
| `/orchestrate` | フルワークフロー実行 |
| `/dispatch` | タスク投入のみ |
| `/harvest` | PR収集・レポート生成 |

---

## Key Reminders

- **`&` は非公式** - 動作しなければworktreeへ
- **ブランチ保護必須** - auto-mergeの前提条件
- **scope分離** - 並列作業の生命線
- **小さく分ける** - 1PR < 500行が理想

---

## Evidence Index

| ID | Source | Date | Excerpt |
|----|--------|------|---------|
| [E1] | docs/boris_fullauto_cli_web_parallel | 2026-01-05 | "自動化の核心はゲートの自動化" |
| [E2] | docs/boris_fullauto_cli_web_parallel | 2026-01-05 | "scope.excludeを明示（同時編集を避ける）" |
| [E3] | docs/boris_fullauto_cli_web_parallel | 2026-01-05 | "小さく分けたPR" |

---

## References

- [plan-format.md](references/plan-format.md) - タスクカードYAML仕様
- [github-setup.md](references/github-setup.md) - GitHub設定ガイド
- [dispatch-methods.md](references/dispatch-methods.md) - 投入方式詳細
- [troubleshooting.md](references/troubleshooting.md) - よくある問題
