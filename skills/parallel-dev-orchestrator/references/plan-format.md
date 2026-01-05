# Plan Format Specification

タスクカード（plan.yaml）のYAMLスキーマ定義。

---

## Full Schema

```yaml
# 必須フィールド
epic: string              # 大目的（1行で説明）
base_branch: string       # ベースブランチ（通常 "main"）
timestamp: string         # 実行タイムスタンプ（YYYYMMDD-HHMM）

tasks:                    # タスク配列（2-5個推奨）
  - id: string            # タスクID（T01, T02, ...）
    title: string         # タスク名（簡潔に）
    branch: string        # ブランチ名（cc/{timestamp}/{id}-{slug}）
    scope:
      include: string[]   # 触っていいパス
      exclude: string[]   # 触らないパス（重要）
    done: string[]        # 完了条件（テスト含む）
    risk: enum            # low | medium | high
    dependencies: string[] # 依存タスクID（なければ空配列）

# オプションフィールド
merge_order: string[]     # マージ順序（依存関係考慮）
verification:
  pre_merge: string[]     # マージ前検証コマンド
  post_merge: string[]    # マージ後検証コマンド
notes: string             # 補足メモ
```

---

## Field Details

### epic

```yaml
epic: "ユーザー認証をOAuth2対応にする"
```

- 1行で全体目的を説明
- 曖昧すぎず、具体的に

### timestamp

```yaml
timestamp: "20260105-1400"
```

- 形式: `YYYYMMDD-HHMM`
- ブランチ名の接頭辞として使用
- 同一epicの識別子

### tasks[].id

```yaml
id: T01
```

- 形式: `T` + 2桁数字（大文字）
- 連番で付与
- merge_order等で参照

### 命名正規化ルール

| 場所 | 形式 | 例 |
|------|------|-----|
| `tasks[].id` | 大文字 `T` + 2桁数字 | `T01`, `T02` |
| ブランチ名 | 小文字 `t` + 2桁数字 | `cc/.../t01-oauth` |
| worktree名 | 小文字 `t` + 2桁数字 | `.worktrees/t01` |
| PR検索 | 小文字で検索 | `gh pr list --search "t01"` |
| ログ出力 | 大文字で表示 | `T01: OAuth2追加` |

```bash
# ID → ブランチ名変換
TASK_ID="T01"
BRANCH_ID=$(echo "$TASK_ID" | tr 'A-Z' 'a-z')  # → t01

# ブランチ名 → ID変換
BRANCH="cc/20260105-1400/t01-oauth"
TASK_ID=$(echo "$BRANCH" | sed -E 's/.*\/(t[0-9]+)-.*/\U\1/')  # → T01
```

> **原則**: plan.yaml内は大文字、ファイルシステム/Git は小文字

### tasks[].branch

```yaml
branch: "cc/20260105-1400/t01-oauth2-provider"
```

- 形式: `cc/{timestamp}/{id}-{slug}`
- `cc/` プレフィックス = Claude Code生成
- slug は短く（3-4単語）

### tasks[].scope

```yaml
scope:
  include:
    - "src/auth/"
    - "config/oauth.ts"
  exclude:
    - "src/auth/legacy/"
    - "src/session/"
```

- **include**: このタスクで触るパス
- **exclude**: 絶対に触らないパス

> **重要**: excludeは並列タスク間のコンフリクト防止の生命線

### tasks[].done

```yaml
done:
  - "OAuth2ログインがGoogle対応"
  - "npm test -- auth 通過"
  - "Lint警告なし"
```

- 検証可能な完了条件
- テストコマンドを含める
- 曖昧な表現を避ける

### tasks[].risk

```yaml
risk: medium
```

| 値 | 基準 |
|----|------|
| `low` | 変更範囲が狭い、既存テストで検証可能 |
| `medium` | 複数ファイル変更、新規テスト必要 |
| `high` | 破壊的変更、マイグレーション必要 |

### tasks[].dependencies

```yaml
dependencies: ["T01"]
```

- このタスクが依存する先行タスクID
- 依存がなければ空配列 `[]`
- 循環依存は禁止

### merge_order

```yaml
merge_order: ["T01", "T02", "T03"]
```

- マージする順序
- 依存関係を考慮
- 省略時はID順

### verification

```yaml
verification:
  pre_merge:
    - "npm test"
    - "npm run lint"
  post_merge:
    - "npm run e2e"
```

- `pre_merge`: 各PRマージ前に実行
- `post_merge`: 全PRマージ後に実行

---

## Validation Rules

### 必須チェック

- [ ] `epic` が存在する
- [ ] `base_branch` が存在する
- [ ] `timestamp` が正しい形式
- [ ] `tasks` が1つ以上ある
- [ ] 各タスクに `id`, `title`, `branch`, `scope`, `done` がある

### 整合性チェック

- [ ] task.id が重複していない
- [ ] branch名が重複していない
- [ ] dependencies のIDが存在する
- [ ] 循環依存がない
- [ ] merge_order のIDが全て存在する

### 品質チェック

- [ ] タスク数が2-5個（推奨）
- [ ] 各タスクのdoneに検証コマンドがある
- [ ] scope.exclude が明示されている
- [ ] 1タスクの変更が500行以下（推奨）

---

## Example: Complete Plan

```yaml
epic: "認証システムをOAuth2 + Redis対応に"
base_branch: "main"
timestamp: "20260105-1400"

tasks:
  - id: T01
    title: "OAuth2プロバイダー追加"
    branch: "cc/20260105-1400/t01-oauth2"
    scope:
      include:
        - "src/auth/providers/"
        - "config/oauth.ts"
      exclude:
        - "src/auth/session/"
        - "src/auth/legacy/"
    done:
      - "GoogleログインがOAuth2で動作"
      - "npm test -- auth/providers 通過"
    risk: medium
    dependencies: []

  - id: T02
    title: "セッションストアRedis移行"
    branch: "cc/20260105-1400/t02-redis"
    scope:
      include:
        - "src/auth/session/"
        - "config/redis.ts"
      exclude:
        - "src/auth/providers/"
    done:
      - "サーバー再起動後もセッション維持"
      - "npm test -- auth/session 通過"
    risk: low
    dependencies: []

  - id: T03
    title: "認証フロー統合テスト"
    branch: "cc/20260105-1400/t03-e2e"
    scope:
      include:
        - "tests/e2e/auth/"
      exclude:
        - "src/"
    done:
      - "E2Eテスト全通過"
      - "npm run e2e -- auth 通過"
    risk: low
    dependencies: ["T01", "T02"]

merge_order: ["T01", "T02", "T03"]

verification:
  pre_merge:
    - "npm test"
    - "npm run lint"
  post_merge:
    - "npm run e2e"

notes: |
  T01とT02は並列実行可能。
  T03はT01, T02完了後に実行。
```

---

## Anti-patterns

| パターン | 問題 | 対策 |
|---------|------|------|
| scope重複 | コンフリクト確定 | excludeで明示的に分離 |
| 曖昧なdone | 完了判定不能 | テストコマンドを含める |
| 大きすぎるタスク | レビュー困難 | 500行以下に分割 |
| 依存関係無視 | マージ失敗 | dependencies明記 |
