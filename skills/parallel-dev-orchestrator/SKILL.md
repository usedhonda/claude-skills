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

1. **Plan**: epicをタスク分解 → `reports/plan.yaml`
2. **Dispatch**: 各タスクをWeb or worktreeに投入
3. **Harvest**: PR監視 → auto-merge → レポート生成

---

## When to Use

- 複数の独立した作業を並列で進めたい
- 1つの大きな機能を分割して開発したい
- Claude Code Webも活用して開発速度を上げたい

**Trigger phrases**: `/orchestrate`, `並列開発`, `タスク分解`, `parallel tasks`

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
