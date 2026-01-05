# Dispatch Methods

タスクをClaude Code Web/CLIに投入する方式の詳細。

---

## Overview

| 方式 | 自動度 | 信頼性 | 備考 |
|------|--------|--------|------|
| **& 方式（Boris式）** | 高 | 低 | 公式未ドキュメント |
| **Worktree方式** | 中 | 高 | 確実だが手動起動 |
| **手動Web** | 低 | 高 | 最終手段 |

---

## Method 1: & 方式（Boris式）

### 概要

プロンプト末尾に `&` を付けることで、タスクをClaude Code Webに投入。

> **注意**: 公式ドキュメントに記載なし。動作しない場合は他方式へ。

### 使い方

```
T01: OAuth2プロバイダー追加

Branch: cc/20260105-1400/t01-oauth2
Base: main

Scope:
- include: src/auth/providers/, config/oauth.ts
- exclude: src/auth/session/

Done:
- GoogleログインがOAuth2で動作
- npm test -- auth/providers 通過

Rules:
- excludeパスは触らない
- PR作成: gh pr create --title "T01: OAuth2プロバイダー追加"

Start now. &
```

### 期待される動作

1. Webセッションが開始
2. 指定ブランチで作業
3. 完了後にPR作成

### 失敗パターン

- `&` が無視される → Worktree方式へ
- Webに接続されない → 手動Web投入
- ブランチが切り替わらない → ブランチを事前push確認

### 自動フォールバック条件

`&` 方式から Worktree へ自動切替する判定基準：

| 条件 | 判定方法 | タイムアウト |
|------|----------|--------------|
| PRが作成されない | `gh pr list --search "cc/$TIMESTAMP"` | 5分 |
| ブランチに変更なし | `git log origin/$BRANCH --since="5 min ago"` | 5分 |
| Webセッション不明 | 手動確認が必要 | - |

```bash
# フォールバック判定スクリプト
TIMESTAMP="20260105-1400"
TASK_ID="t01"
TIMEOUT_MIN=5

# N分待機
sleep $((TIMEOUT_MIN * 60))

# PRが見つかるか確認
PR_COUNT=$(gh pr list --search "cc/$TIMESTAMP/$TASK_ID" --json number -q 'length')

if [ "$PR_COUNT" = "0" ]; then
  echo "⚠️ & 方式タイムアウト → Worktree へフォールバック"
  # Worktree 方式を実行
fi
```

> **推奨**: 初回は5分、信頼性確認後は3分に短縮

---

## Method 2: Worktree方式

### 概要

Git worktreeで各タスク用の作業ディレクトリを作成し、別ターミナルでCLI起動。

### 使い方

#### Step 1: Worktree作成

```bash
# 各タスクについて
git worktree add .worktrees/t01 cc/20260105-1400/t01-oauth2
git worktree add .worktrees/t02 cc/20260105-1400/t02-redis
```

#### Step 2: タスクプロンプト配置

```bash
# テンプレートからプロンプト生成
cat > .worktrees/t01/.claude-task.md << 'EOF'
T01: OAuth2プロバイダー追加

Branch: cc/20260105-1400/t01-oauth2
...
EOF
```

#### Step 3: 別ターミナルで起動

```bash
# ターミナル1
cd .worktrees/t01
claude
# → .claude-task.md の内容を貼り付け

# ターミナル2
cd .worktrees/t02
claude
# → 同様に
```

### 利点

- 確実に動作
- 各セッションが完全分離
- ローカルで全作業を追跡可能

### 欠点

- 手動でターミナル起動が必要
- ディスク容量を使用

### クリーンアップ

```bash
# 作業完了後
git worktree remove .worktrees/t01
git worktree remove .worktrees/t02
git worktree prune
```

---

## Method 3: 手動Web投入

### 概要

claude.ai/code を直接開いてタスクを投入。

### 使い方

1. https://claude.ai/code を開く
2. 対象リポジトリを選択
3. タスクプロンプトを貼り付け
4. ブランチを指定して実行

### 利点

- 最も確実
- UIで進捗確認可能

### 欠点

- 完全手動
- 複数タスクの管理が煩雑

---

## Fallback Strategy

```
┌──────────────────┐
│  & 方式を試行    │
└────────┬─────────┘
         │
    動作した？
         │
    Yes ─┼─ No
         │    │
         ▼    ▼
┌────────────┐  ┌──────────────────┐
│  完了      │  │ Worktree方式へ   │
└────────────┘  └────────┬─────────┘
                         │
                    動作した？
                         │
                    Yes ─┼─ No
                         │    │
                         ▼    ▼
               ┌────────────┐  ┌──────────────────┐
               │  完了      │  │ 手動Web投入      │
               └────────────┘  └──────────────────┘
```

---

## 自動選択ロジック

```bash
#!/bin/bash
# dispatch_task.sh

TASK_PROMPT=$1
METHOD=${2:-"auto"}

dispatch_ampersand() {
  echo "$TASK_PROMPT &"
  # &の結果を待つ（タイムアウト30秒）
  sleep 30
  # Web接続確認
  # ...
}

dispatch_worktree() {
  TASK_ID=$(echo "$TASK_PROMPT" | grep -oP 'T\d+')
  BRANCH=$(echo "$TASK_PROMPT" | grep -oP 'Branch: \K[^\n]+')

  git worktree add ".worktrees/$TASK_ID" "$BRANCH"
  echo "$TASK_PROMPT" > ".worktrees/$TASK_ID/.claude-task.md"

  echo "別ターミナルで実行:"
  echo "cd .worktrees/$TASK_ID && claude"
}

case $METHOD in
  "auto")
    dispatch_ampersand || dispatch_worktree
    ;;
  "&")
    dispatch_ampersand
    ;;
  "worktree")
    dispatch_worktree
    ;;
esac
```

---

## Comparison Table

| 観点 | & 方式 | Worktree | 手動Web |
|------|--------|----------|---------|
| 自動化 | ◎ | △ | ✗ |
| 信頼性 | △ | ◎ | ◎ |
| 並列数 | 無制限? | ディスク依存 | 無制限 |
| 追跡性 | △ | ◎ | △ |
| セットアップ | なし | git worktree | ブラウザ |

---

## Recommendations

### 初回利用時

1. まず`&`を試す
2. 動作しなければWorktreeへ
3. Worktreeの手順を覚えておく

### 安定運用時

- 信頼性重視 → Worktree
- 速度重視 → & (動作確認済みなら)
- 緊急時 → 手動Web

### 大規模並列時

- Worktree推奨
- ディスク容量を事前確認
- 完了後のクリーンアップを忘れずに
