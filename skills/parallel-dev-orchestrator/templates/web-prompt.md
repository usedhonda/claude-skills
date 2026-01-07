# Web Prompt Template

Webセッション用プロンプトのテンプレート。`par-plan`で自動生成される。

## Template

```
# Session {N}: {task_id}

**Copy the content below / 下の枠内をコピー**

━━━━━━━━━━ ✂ COPY START ✂ ━━━━━━━━━━

Repository: {repo_name}

{task_description}

Scope:
- Edit: {include_paths}
- Do NOT edit: {exclude_paths}

Done when:
{done_criteria}

━━━━━━━━━━ ✂ COPY END ✂ ━━━━━━━━━━

⚠️ After completion: Click "Create PR" button / 完了後: PRボタンをクリック
```

## Example

```
# Session 1: T01

**Copy the content below / 下の枠内をコピー**

━━━━━━━━━━ ✂ COPY START ✂ ━━━━━━━━━━

Repository: myapp

Add OAuth2 provider (Google).

Scope:
- Edit: src/auth/providers/, config/oauth.ts
- Do NOT edit: src/auth/session/, tests/e2e/

Done when:
- Google login works with OAuth2
- npm test -- auth/providers passes

━━━━━━━━━━ ✂ COPY END ✂ ━━━━━━━━━━

⚠️ After completion: Click "Create PR" button / 完了後: PRボタンをクリック
```
