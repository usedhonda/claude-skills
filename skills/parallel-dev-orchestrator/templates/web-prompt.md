# Web Prompt Template

Webセッション用プロンプトのテンプレート。`par-plan`で自動生成される。

## Template

```
# Session {N}: {task_id}

**Copy the content below / 下の枠内をコピー**

━━━━━━━━━━ ✂ COPY START ✂ ━━━━━━━━━━

Repository: {repo_name}
Branch: {branch_name}

{task_description}

Scope:
- Edit: {include_paths}
- Do NOT edit: {exclude_paths}

Done when:
{done_criteria}

Create PR when done.

━━━━━━━━━━ ✂ COPY END ✂ ━━━━━━━━━━
```

## Example

```
# Session 1: T01

**Copy the content below / 下の枠内をコピー**

━━━━━━━━━━ ✂ COPY START ✂ ━━━━━━━━━━

Repository: myapp
Branch: cc/20260107-1500/t01-oauth2

Add OAuth2 provider (Google).

Scope:
- Edit: src/auth/providers/, config/oauth.ts
- Do NOT edit: src/auth/session/, tests/e2e/

Done when:
- Google login works with OAuth2
- npm test -- auth/providers passes

Create PR when done.

━━━━━━━━━━ ✂ COPY END ✂ ━━━━━━━━━━
```
