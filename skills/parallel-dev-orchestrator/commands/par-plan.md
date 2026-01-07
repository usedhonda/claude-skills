---
description: Create parallel task plan from goal
argument-hint: "goal description"
---

# par-plan

ゴールをタスクに分解し、ブランチ作成からWebプロンプト生成まで実行。

## Usage

```
/parallel-dev-orchestrator:par-plan "OAuth2認証を追加"
```

## Process

### Step 1: Analyze & Plan

1. ゴールを分析
2. 2-5個のタスクに分解
3. 各タスクのscope（include/exclude）を定義
4. `reports/plan-{timestamp}.yaml` に保存

### Step 2: Confirm with User

計画をテーブル形式で表示し、確認を求める:

```
## Plan Summary

Goal: OAuth2認証を追加
Tasks: 3

| ID | Title | Scope | Risk |
|----|-------|-------|------|
| T01 | OAuth2プロバイダー | src/auth/providers/ | medium |
| T02 | セッションRedis | src/auth/session/ | low |
| T03 | 認証E2Eテスト | tests/e2e/auth/ | low |

この計画で進めますか?
```

ユーザーが承認したら次へ。修正点があれば反映。

### Step 3: Create Branches

**Claudeが自動実行:**

```bash
git checkout -b cc/{timestamp}/t01-xxx
git push -u origin cc/{timestamp}/t01-xxx
git checkout main
# 全タスク分繰り返し
```

### Step 4: Generate Web Prompts

各タスク用のWebセッションプロンプトを生成・表示:

```
## Web Session Prompts

以下を claude.ai/code の各セッションに貼り付けてください:

---
### T01: OAuth2プロバイダー追加

リポジトリ: {repo}
ブランチ: cc/{timestamp}/t01-oauth2

## タスク
OAuth2プロバイダーを追加

## Scope
- 触ってよい: src/auth/providers/, config/oauth.ts
- 触らない: src/auth/session/

## Done
- GoogleログインがOAuth2で動作
- npm test -- auth/providers 通過

完了したら git add/commit/push して gh pr create

---
### T02: ...
```

## Output Format

```yaml
goal: "OAuth2認証を追加"
base_branch: "main"
timestamp: "20260106-1500"

tasks:
  - id: T01
    title: "OAuth2プロバイダー"
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
```

## Scope Rules

| Rule | Description |
|------|-------------|
| Scope isolation | タスク間でファイル重複なし |
| Exclude required | 他タスクのincludeはexcludeに |
| Verifiable done | テストコマンドを含める |
| Risk level | `high` = auto-merge禁止 |

## Complete Output

```
Plan: reports/plan-20260106-1500.yaml

Branches created:
  cc/20260106-1500/t01-oauth2 (pushed)
  cc/20260106-1500/t02-redis (pushed)

Web Prompts:
  [上記プロンプト表示]

PRが作成されたら /parallel-dev-orchestrator:par-harvest
```

## See Also

- [par-dispatch](./par-dispatch.md) - Worktree方式で実行する場合
- [par-harvest](./par-harvest.md) - PRをマージ
