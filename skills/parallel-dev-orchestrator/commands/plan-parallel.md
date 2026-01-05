---
description: 並列開発前提の計画モード。plan.yaml形式で出力
argument-hint: [epic説明]
---

# Plan for Parallel Development

並列開発を前提とした計画を立てます。通常の `/plan` と異なり、計画ファイルに `plan.yaml` 形式を含めます。

## 使い方

```
/plan-parallel "ユーザー認証をOAuth2対応にする"
```

## 計画時の指針

### 1. タスク分解の原則

- **2-5個のタスク**に分解
- **1タスク = 1ブランチ = 1PR**
- **scope.exclude**で同時編集を防ぐ

### 2. 出力形式

計画ファイルに以下の YAML ブロックを含めてください：

```yaml
# Parallel Development Plan

epic: "{ユーザーの説明}"
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
    risk: low  # low | medium | high
    dependencies: []

  - id: T02
    title: "タスク名"
    branch: "cc/{timestamp}/t02-slug"
    scope:
      include: ["触るパス"]
      exclude: ["T01のinclude"]  # 重要: 他タスクと重複させない
    done:
      - "完了条件"
    risk: low
    dependencies: []

merge_order: ["T01", "T02"]

verification:
  pre_merge: ["npm test", "npm run lint"]
```

### 3. 重要なルール

| ルール | 説明 |
|--------|------|
| scope分離 | タスク間で触るファイルを重複させない |
| excludeは必須 | 他タスクのincludeをexcludeに |
| doneは検証可能に | テストコマンドを含める |
| riskを設定 | high はauto-merge対象外 |

---

## 計画承認後の実行

計画が承認されたら：

```bash
/orchestrate --from-plan
```

これにより：
1. 計画ファイルから YAML ブロックを抽出
2. `reports/plan-{timestamp}.yaml` に保存
3. Dispatch → Monitor → Harvest を実行

---

## 例: OAuth2対応の計画

```yaml
# Parallel Development Plan

epic: "ユーザー認証をOAuth2対応にする"
base_branch: "main"
timestamp: "20260106-1500"

tasks:
  - id: T01
    title: "OAuth2プロバイダー追加"
    branch: "cc/20260106-1500/t01-oauth2"
    scope:
      include: ["src/auth/providers/", "config/oauth.ts"]
      exclude: ["src/auth/session/"]
    done:
      - "GoogleログインがOAuth2で動作"
      - "npm test -- auth/providers 通過"
    risk: medium
    dependencies: []

  - id: T02
    title: "セッションストアRedis移行"
    branch: "cc/20260106-1500/t02-redis"
    scope:
      include: ["src/auth/session/", "config/redis.ts"]
      exclude: ["src/auth/providers/"]
    done:
      - "サーバー再起動後もセッション維持"
      - "npm test -- auth/session 通過"
    risk: low
    dependencies: []

merge_order: ["T01", "T02"]

verification:
  pre_merge: ["npm test", "npm run lint"]
```

---

## 関連コマンド

- `/orchestrate` - フルワークフロー実行
- `/orchestrate --from-plan` - この計画から実行
- `/plan` - 通常の計画モード（並列開発なし）
