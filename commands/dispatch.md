---
description: 既存プランからタスクを投入（& or worktree）
argument-hint: [--plan path] [--task T01] [--method &|worktree|web]
---

# Dispatch Tasks

既存の`plan.yaml`からタスクをClaude Code Web/CLIに投入。

## 現在の状態

```bash
# 既存プラン一覧
!ls -la reports/plan-*.yaml 2>/dev/null || echo "No plans found"

# 現在のworktree
!git worktree list
```

## 使い方

### 全タスク投入

```
/dispatch
/dispatch --plan reports/plan-20260105-1400.yaml
```

### 特定タスクのみ

```
/dispatch --task T01
/dispatch --task T02 --method worktree
```

### 方式指定

```
/dispatch --method &          # Boris式（デフォルト）
/dispatch --method worktree   # Worktree方式
/dispatch --method web        # 手動Web（手順表示のみ）
```

---

## 実行フロー

### 1. プラン読み込み

```bash
# 最新のplan.yamlを使用
PLAN=$(ls -t reports/plan-*.yaml | head -1)
```

### 2. ブランチ確認

```bash
# 各タスクのブランチが存在するか確認
for branch in $(yq '.tasks[].branch' $PLAN); do
  git rev-parse --verify origin/$branch || echo "Missing: $branch"
done
```

### 3. タスクプロンプト生成

templates/task-prompt.md.template を使用:

```
{TASK_ID}: {TASK_TITLE}

Branch: {BRANCH_NAME}
Base: {BASE_BRANCH}

Scope:
- include: {SCOPE_INCLUDE}
- exclude: {SCOPE_EXCLUDE}

Done:
{DONE_CRITERIA}

Rules:
1. excludeパスは触らない
2. 変更は500行以下に
3. 完了後: gh pr create --title "{TASK_ID}: {TASK_TITLE}"

Start now.
```

### 4. 投入実行

#### & 方式

```
{generated_prompt} &
```

#### Worktree方式

```bash
git worktree add .worktrees/{task_id} {branch}
echo "{generated_prompt}" > .worktrees/{task_id}/.claude-task.md

echo "別ターミナルで実行:"
echo "cd .worktrees/{task_id} && claude"
```

#### Web方式

```
以下をclaude.ai/codeに貼り付けてください:

---
{generated_prompt}
---

ブランチ: {branch}
```

---

## オプション詳細

### --plan

使用するプランファイルを指定。省略時は最新を使用。

```
/dispatch --plan reports/plan-20260105-1400.yaml
```

### --task

特定タスクのみ投入。複数指定可能。

```
/dispatch --task T01
/dispatch --task T01 --task T02
```

### --method

投入方式を指定。

| 値 | 動作 |
|----|------|
| `&` | Boris式（デフォルト） |
| `worktree` | Git worktree作成 |
| `web` | 手順表示のみ |

---

## 出力例

### & 方式

```
Dispatching T01: OAuth2プロバイダー追加

T01: OAuth2プロバイダー追加

Branch: cc/20260105-1400/t01-oauth2
Base: main
...

Start now. &

---

T01 dispatched via &
Waiting for Web session...
```

### Worktree方式

```
Dispatching T01: OAuth2プロバイダー追加

Creating worktree...
git worktree add .worktrees/t01 cc/20260105-1400/t01-oauth2

Task prompt saved to: .worktrees/t01/.claude-task.md

To start, run in new terminal:
  cd .worktrees/t01 && claude

---

Dispatching T02: セッションRedis移行

Creating worktree...
git worktree add .worktrees/t02 cc/20260105-1400/t02-redis

Task prompt saved to: .worktrees/t02/.claude-task.md

To start, run in new terminal:
  cd .worktrees/t02 && claude
```

---

## 関連コマンド

- `/orchestrate` - フルワークフロー
- `/harvest` - PR収集・レポート
