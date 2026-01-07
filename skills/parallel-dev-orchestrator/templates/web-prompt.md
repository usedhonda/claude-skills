# Web Prompt Template

Webセッション用プロンプトのテンプレート。`par-plan`で自動生成される。

## Template

```
{task_id}: {task_title}

リポジトリ: {repo_name}
ブランチ: {branch_name}

## タスク

{task_description}

## Scope

触ってよいファイル:
{include_paths}

触らないファイル:
{exclude_paths}

## Done Criteria

{done_criteria}

## Rules

1. excludeに記載されたパスは絶対に変更しない
2. 完了したら以下を実行:

git add .
git commit -m "{task_id}: {task_title}"
git push
gh pr create --title "{task_id}: {task_title}" --body "Parallel task PR"
```

## Example

```
T01: OAuth2プロバイダー追加

リポジトリ: myapp
ブランチ: cc/20260107-1500/t01-oauth2

## タスク

OAuth2プロバイダー(Google)を追加してください。

## Scope

触ってよいファイル:
- src/auth/providers/
- config/oauth.ts

触らないファイル:
- src/auth/session/
- tests/e2e/

## Done Criteria

- GoogleログインがOAuth2で動作すること
- npm test -- auth/providers が通過すること

## Rules

1. excludeに記載されたパスは絶対に変更しない
2. 完了したら以下を実行:

git add .
git commit -m "T01: OAuth2プロバイダー追加"
git push
gh pr create --title "T01: OAuth2プロバイダー追加" --body "Parallel task PR"
```
