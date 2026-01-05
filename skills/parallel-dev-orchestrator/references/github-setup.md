# GitHub Setup Guide

並列開発オーケストレーションの前提条件設定。

---

## Overview

auto-mergeを安全に動作させるには以下が必須:

1. **Allow auto-merge** - リポジトリ設定
2. **Branch protection** - mainブランチ保護
3. **GitHub CLI** - gh認証

---

## 1. Allow Auto-merge

### 設定手順

1. GitHub → Repository → Settings
2. General → Pull Requests セクション
3. **Allow auto-merge** にチェック

```
Settings → General → Pull Requests
└── ✓ Allow auto-merge
```

### 確認コマンド

```bash
gh api repos/{owner}/{repo} --jq '.allow_auto_merge'
# true なら有効
```

---

## 2. Branch Protection Rules

### 必須設定

```
Settings → Branches → Add branch protection rule

Branch name pattern: main

✓ Require a pull request before merging
  └── Required approving reviews: 1 (オプション)

✓ Require status checks to pass before merging
  └── ✓ Require branches to be up to date before merging
  └── Status checks: (CI名を追加)

✓ Do not allow bypassing the above settings
```

### 推奨設定

```
□ Require signed commits (オプション)
✓ Require linear history (推奨)
□ Allow force pushes (無効のまま)
□ Allow deletions (無効のまま)
```

### 確認コマンド

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --jq '{
    required_pull_request_reviews: .required_pull_request_reviews,
    required_status_checks: .required_status_checks
  }'
```

---

## 3. GitHub CLI Setup

### インストール

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Windows
winget install GitHub.cli
```

### 認証

```bash
gh auth login
# → GitHub.com を選択
# → HTTPS を選択
# → ブラウザで認証

# 確認
gh auth status
```

### 必要なスコープ

```
repo (Full control of private repositories)
├── repo:status
├── repo_deployment
├── public_repo
├── repo:invite
└── security_events
```

---

## 4. CI/CD Setup (Required Checks)

### GitHub Actions例

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
      - run: npm run lint
```

### ステータスチェック登録

1. 一度PRを作成してCIを走らせる
2. Settings → Branches → main → Edit
3. "Require status checks" で `test` を検索・追加

---

## 5. Verification Checklist

### Pre-flight Check Script

```bash
#!/bin/bash
# scripts/check-github-setup.sh

REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
echo "Checking: $REPO"

# Auto-merge
AUTO_MERGE=$(gh api repos/$REPO --jq '.allow_auto_merge')
if [ "$AUTO_MERGE" = "true" ]; then
  echo "✓ Auto-merge: enabled"
else
  echo "✗ Auto-merge: disabled"
  echo "  → Settings → General → Allow auto-merge"
fi

# Branch protection
PROTECTION=$(gh api repos/$REPO/branches/main/protection 2>/dev/null)
if [ -n "$PROTECTION" ]; then
  echo "✓ Branch protection: configured"

  # Required checks
  CHECKS=$(echo $PROTECTION | jq -r '.required_status_checks.contexts[]?' 2>/dev/null)
  if [ -n "$CHECKS" ]; then
    echo "  Required checks: $CHECKS"
  else
    echo "  ⚠ No required status checks"
  fi
else
  echo "✗ Branch protection: not configured"
  echo "  → Settings → Branches → Add rule for 'main'"
fi

# GH auth
if gh auth status &>/dev/null; then
  echo "✓ GitHub CLI: authenticated"
else
  echo "✗ GitHub CLI: not authenticated"
  echo "  → Run: gh auth login"
fi
```

### 実行

```bash
chmod +x scripts/check-github-setup.sh
./scripts/check-github-setup.sh
```

---

## Troubleshooting

### "Auto-merge is not allowed"

```
Error: Pull request is not mergeable: Auto-merge is not allowed for this repository
```

**原因**: リポジトリ設定でauto-merge無効

**解決**: Settings → General → Allow auto-merge ✓

### "Required status check is expected"

```
Error: Required status check "test" is expected
```

**原因**: CIがまだ走っていない or 名前が違う

**解決**:
1. CIを一度走らせる
2. ステータスチェック名を確認
3. Branch protection で正しい名前を設定

### "Review required"

```
Error: At least 1 approving review is required
```

**原因**: レビュー必須設定

**解決（ソロ開発の場合）**:
- Required approving reviews を 0 に
- または自分でApproveする

---

## Quick Reference

| 設定 | 場所 | 必須 |
|------|------|------|
| Allow auto-merge | Settings → General | Yes |
| Branch protection | Settings → Branches | Yes |
| Required checks | Branch protection rule | Yes |
| Required reviews | Branch protection rule | Optional |
| gh auth | ローカル | Yes |
