---
name: parallel-dev-orchestrator
description: |
  並列開発ワークフロー。
  タスク分解→ブランチ作成→並列投入→auto-merge→レポート。
  "/parallel-dev-orchestrator:par-plan", "並列開発", "タスク分解"で発動。
compression-anchors:
  - "& でWeb投入、失敗時はworktreeフォールバック"
  - "Plan→Dispatch→Monitor→Harvest"
  - "1タスク = 1ブランチ = 1PR"
agents:
  - worktree-dispatcher
  - parallel-planner
---

# Parallel Dev Orchestrator

CLI⇄Web並列開発を自動化するBoris流ワークフロー。

## TL;DR

```
/par-plan "ゴール" → 計画確認 → ブランチ自動作成 → Webプロンプト表示
                                        ↓
                              claude.ai/code に貼り付けて実行
                                        ↓
                              /par-harvest でPR収穫
```

1. **Plan**: ゴールをタスク分解 → 確認 → ブランチ作成 → プロンプト生成
2. **Execute**: Webセッションでプロンプトを実行（並列可）
3. **Harvest**: PRをマージ → レポート生成

---

## When to Use

- 複数の独立した作業を並列で進めたい
- 1つの大きな機能を分割して開発したい
- Claude Code Webも活用して開発速度を上げたい

**Trigger phrases**: `/parallel-dev-orchestrator:par-plan`, `並列開発`, `タスク分解`, `parallel tasks`

---

## When NOT to Use

| ケース | 理由 | 代替案 |
|--------|------|--------|
| 大規模リネーム/リファクタ | 複数ブランチで同一ファイルを触る | 単一ブランチで実行 |
| 広域の設計変更 | scope分離が困難 | 先に設計PRを1つ作成 |
| DBマイグレーション | 実行順序が厳密 | 依存順に直列実行 |
| 同一ファイル集合を触るタスク | コンフリクト確定 | タスクを再分割 |

**判断基準**: scope.excludeで明確に分離できないなら、並列化しない。

---

## Risk Policy

| Risk | auto-merge | 必須条件 |
|------|------------|----------|
| **low** | ✅ 許可 | CI通過 |
| **medium** | ✅ 許可 | CI通過 + required checks |
| **high** | ❌ 禁止 | 手動レビュー必須 |

**High riskの例**: 認証/決済/データ削除/外部API連携/本番設定変更

---

## Prerequisites

1. **GitHub Auto-merge有効化**: Settings → General → Allow auto-merge ✓
2. **ブランチ保護ルール設定**: main への直push禁止、status checks必須
3. **GitHub CLI認証**: `gh auth login`
4. **必須ツール**: `yq`, `jq`

詳細: [references/github-setup.md](references/github-setup.md)

---

## Core Workflow

```
┌─────────────────────────────────────────────────────────┐
│  /par-plan "認証機能をOAuth2対応に"                      │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 1: Analyze & Confirm                             │
│  ゴール分析 → タスク分解 → 計画表示 → ユーザー確認        │
│  → reports/plan-{timestamp}.yaml                        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 2: Prompt Generation                             │
│  各タスク用Webプロンプトを自動生成・表示                  │
│  → claude.ai/code に貼り付けて実行                       │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 3: Harvest                                       │
│  /par-harvest → auto-merge有効化 → レポート生成          │
│  → reports/harvest-{timestamp}.md                       │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Start

```bash
# 1. 環境チェック
/parallel-dev-orchestrator:par-init

# 2. 計画作成 → ブランチ作成 → プロンプト生成
/parallel-dev-orchestrator:par-plan "ユーザー認証をOAuth2対応にする"

# 3. Web実行（表示されたプロンプトを claude.ai/code へ）

# 4. PRマージ
/parallel-dev-orchestrator:par-harvest --watch
```

### Alternative: Worktree方式

```bash
/parallel-dev-orchestrator:par-dispatch              # 全タスク
/parallel-dev-orchestrator:par-dispatch --task T01   # 特定タスクのみ
```

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
    scope:
      include: ["src/auth/", "config/oauth.ts"]
      exclude: ["src/auth/legacy/"]
    done:
      - "OAuth2ログインがGoogle対応"
      - "npm test -- auth 通過"
    risk: medium
    dependencies: []
merge_order: ["T01", "T02"]
```

**重要原則**:
- **1タスク = 1ブランチ = 1PR**
- **scope.exclude**: 同時編集を防ぐ（コンフリクト回避）

詳細: [references/plan-format.md](references/plan-format.md)

---

## Constraints

| パターン | 重大度 | 理由 |
|---------|--------|------|
| ブランチ保護なしでauto-merge | Critical | マージ事故の原因 |
| scope重複のまま並列実行 | Critical | コンフリクト確定 |
| 大きすぎるタスク（>500行） | Warning | レビュー困難 |
| テストなしでマージ | Warning | 品質低下 |

---

## Recovery

問題発生時は [references/troubleshooting.md](references/troubleshooting.md) を参照。

主なシナリオ:
- Dispatch失敗 → worktree削除 → 再投入
- PRコンフリクト → rebase → force-push
- CIエラー → 修正 → push

---

## Artifacts

| ファイル | 説明 | 生成タイミング |
|----------|------|----------------|
| `reports/plan-{timestamp}.yaml` | タスク分解結果 | par-plan |
| `.worktrees/t{N}/` | 並列作業ディレクトリ | par-dispatch |
| `reports/harvest-{timestamp}.md` | 完了レポート | par-harvest |

---

## Commands

| コマンド | 説明 |
|---------|------|
| `/parallel-dev-orchestrator:par-status` | 現在のプラン・PR状況を表示 |
| `/parallel-dev-orchestrator:par-init` | 環境チェック |
| `/parallel-dev-orchestrator:par-plan` | タスク分解 |
| `/parallel-dev-orchestrator:par-dispatch` | タスク投入 |
| `/parallel-dev-orchestrator:par-harvest` | PR収集・マージ・レポート生成 |

---

## Key Reminders

- **`&` は非公式** - 動作しなければworktreeへ
- **ブランチ保護必須** - auto-mergeの前提条件
- **scope分離** - 並列作業の生命線
- **小さく分ける** - 1PR < 500行が理想

---

## References

- [plan-format.md](references/plan-format.md) - タスクカードYAML仕様
- [github-setup.md](references/github-setup.md) - GitHub設定ガイド
- [dispatch-methods.md](references/dispatch-methods.md) - 投入方式詳細
- [troubleshooting.md](references/troubleshooting.md) - よくある問題
