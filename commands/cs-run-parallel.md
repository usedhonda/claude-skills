---
description: claude-skills parallel development (init/plan/dispatch/harvest/status)
argument-hint: init|plan|dispatch|harvest|status [options]
---

# Parallel Development

Boris流 CLI⇄Web 並列開発ワークフロー。

## Usage

```
/cs-run-parallel                  # ヘルプ表示
/cs-run-parallel status           # PR状態
/cs-run-parallel init             # 環境診断
/cs-run-parallel plan "epic"      # タスク計画作成
/cs-run-parallel dispatch         # タスク投入
/cs-run-parallel harvest          # PR収集・マージ
```

---

## Subcommands

### status - PR状態表示

```
/cs-run-parallel status
```

**出力例**:

```
📊 Parallel Development Status

Active Plan: reports/plan-20260106-1500.yaml
├─ Epic: ユーザー認証をOAuth2対応にする
├─ Timestamp: 20260106-1500
└─ Tasks: 3

PRs:
├─ #123 T01: OAuth2プロバイダー追加 [OPEN] ✓ checks
├─ #124 T02: セッションRedis移行 [OPEN] ✓ checks
└─ #125 T03: 認証E2Eテスト [OPEN] ⏳ pending

Blockers:
└─ ⚠️ T03: status checks pending

Next Action:
→ Wait for checks, then /cs-run-parallel harvest
```

---

### init - 環境診断

```
/cs-run-parallel init
```

並列開発に必要な環境をチェック。

**チェック項目**:

```
🔍 Environment Check

GitHub CLI:
├─ gh: ✅ installed (2.40.0)
└─ authenticated: ✅ yes

Repository:
├─ allow_auto_merge: ✅ enabled
├─ branch_protection: ✅ main protected
└─ required_checks: ✅ CI configured

Tools:
├─ yq: ✅ installed
└─ jq: ✅ installed

Result: ✅ Ready for parallel development
```

**修正が必要な場合**:

```
❌ allow_auto_merge: disabled

Fix:
  1. Go to: Settings → General → Pull Requests
  2. Enable "Allow auto-merge"

Or run:
  gh api repos/{owner}/{repo} -X PATCH -f allow_auto_merge=true
```

---

### plan - タスク計画作成

```
/cs-run-parallel plan "ユーザー認証をOAuth2対応にする"
```

**実行プロセス**:

1. Epic を分析
2. 2-5個のタスクに分解
3. 各タスクの scope（include/exclude）を決定
4. `reports/plan-{timestamp}.yaml` に保存

**出力形式**:

```yaml
epic: "ユーザー認証をOAuth2対応にする"
base_branch: "main"
timestamp: "20260106-1500"

tasks:
  - id: T01
    title: "OAuth2プロバイダー追加"
    branch: "cc/20260106-1500/t01-oauth2"
    scope:
      include: ["src/auth/providers/"]
      exclude: ["src/auth/session/"]
    done:
      - "GoogleログインがOAuth2で動作"
      - "npm test -- auth/providers 通過"
    risk: medium
    dependencies: []

merge_order: ["T01", "T02"]

verification:
  pre_merge: ["npm test", "npm run lint"]
```

**重要ルール**:

| ルール | 説明 |
|--------|------|
| scope分離 | タスク間でファイルを重複させない |
| excludeは必須 | 他タスクのincludeをexcludeに |
| doneは検証可能に | テストコマンドを含める |
| riskを設定 | high はauto-merge対象外 |

---

### dispatch - タスク投入

```
/cs-run-parallel dispatch                     # 全タスク投入
/cs-run-parallel dispatch --task T01          # 特定タスクのみ
/cs-run-parallel dispatch --method worktree   # Worktree方式
```

**投入方式**:

| 方式 | コマンド | 説明 |
|------|----------|------|
| & | `--method &` | Boris式（デフォルト） |
| Worktree | `--method worktree` | Git worktree作成 |
| Web | `--method web` | 手順表示のみ |

**Worktree方式の場合**:

```
別ターミナルで実行:
  cd .worktrees/t01 && claude
```

---

### harvest - PR収集・マージ

```
/cs-run-parallel harvest                      # 収集・auto-merge
/cs-run-parallel harvest --watch              # 完了まで待機
/cs-run-parallel harvest --report-only        # レポートのみ
```

**Risk Policy**:

| Risk | Auto-merge |
|------|------------|
| low | ✅ 有効化 |
| medium | ✅ 有効化 |
| high | ❌ 手動レビュー必須 |

**完了レポート**:

```markdown
# Orchestration Report: 20260106-1500

## Summary
| Item | Value |
|------|-------|
| Epic | ユーザー認証をOAuth2対応にする |
| Duration | 2h 30m |

## Tasks
| ID | Title | Status | PR |
|----|-------|--------|-----|
| T01 | OAuth2プロバイダー追加 | DONE | #123 |
| T02 | セッションRedis移行 | DONE | #124 |
```

---

## Quick Start

```
1. /cs-run-parallel init              # 環境チェック
2. /cs-run-parallel plan "epic説明"   # 計画作成
3. /cs-run-parallel dispatch          # タスク投入
4. /cs-run-parallel status            # 進捗確認
5. /cs-run-parallel harvest           # 収集・マージ
```

---

## フルワークフロー実行

計画から収集まで一括実行する場合:

```
/cs-run-parallel plan "epic説明"
# → 計画確認後
/cs-run-parallel dispatch
# → タスク完了後
/cs-run-parallel harvest --watch
```

---

## トラブルシューティング

### & が動作しない

→ Worktree方式にフォールバック:

```bash
/cs-run-parallel dispatch --method worktree
```

### auto-merge有効化失敗

→ GitHub設定確認:

```bash
gh api repos/{owner}/{repo} --jq '.allow_auto_merge'
# false → Settings → General → Allow auto-merge
```

### コンフリクト発生

→ scope.exclude設定を確認、重複があれば分離

---

## 詳細ドキュメント

- [plan-format.md](../skills/parallel-dev-orchestrator/references/plan-format.md)
- [dispatch-methods.md](../skills/parallel-dev-orchestrator/references/dispatch-methods.md)
- [github-setup.md](../skills/parallel-dev-orchestrator/references/github-setup.md)
- [troubleshooting.md](../skills/parallel-dev-orchestrator/references/troubleshooting.md)
