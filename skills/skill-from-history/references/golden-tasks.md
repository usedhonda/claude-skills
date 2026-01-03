# Golden Tasks (Eval Harness)

制約とスキルの有効性を検証するための評価ハーネス仕様。

---

## 概念

> "A learning system is only as smart as its forgetting, its tests, and its ability to explain itself." (ChatGPT)

Golden Task = 成功した過去のインタラクションを再現可能なテストケースとして保存。
新しい制約がこれらのタスクを壊さないことを検証する。

---

## Golden Task 形式

### ファイル構造

```
.claude/golden-tasks/
├── index.yaml           # タスク一覧
├── api-endpoint-creation.yaml
├── security-review.yaml
└── refactoring-pattern.yaml
```

### タスク定義 (YAML)

```yaml
# api-endpoint-creation.yaml
id: GT-001
name: API Endpoint Creation
description: Standard pattern for creating a new Hono API endpoint
created_at: 2025-01-03
tags: [api, hono, endpoint]

# 入力コンテキスト
input:
  prompt: "Create a new API endpoint for user profile"
  files:
    - path: src/routes/index.ts
      content: |
        import { Hono } from 'hono';
        const app = new Hono();
        // existing routes...

# 期待される出力
expected:
  files_modified:
    - path: src/routes/user.ts
      contains:
        - "import { Hono } from 'hono'"
        - "app.get('/profile'"
      not_contains:
        - "import express"
        - "Router()"

  constraints_respected:
    - CONS-001  # Express → Hono
    - CONS-002  # Route naming convention

# 評価基準
criteria:
  - name: Uses Hono
    type: contains
    target: src/routes/user.ts
    pattern: "Hono"
    weight: 1.0

  - name: No Express
    type: not_contains
    target: src/routes/user.ts
    pattern: "express"
    weight: 1.0

# メタデータ
metadata:
  source_session: abc123
  source_commit: a1b2c3d
  verified_by: user
  last_run: 2025-01-02
  pass_rate: 1.0
```

---

## 評価ワークフロー

### 1. タスク実行

```bash
$ /skill-eval

## Running Golden Tasks

| Task | Status | Duration | Notes |
|------|--------|----------|-------|
| GT-001 | PASS | 2.3s | All criteria met |
| GT-002 | FAIL | 1.8s | Constraint CONS-003 violated |
| GT-003 | SKIP | - | Prerequisite not met |

## Summary
- Total: 3
- Passed: 1
- Failed: 1
- Skipped: 1
```

### 2. ベースライン比較

```bash
$ /skill-eval --baseline

## Baseline Comparison

| Metric | Without Skills | With Skills | Delta |
|--------|----------------|-------------|-------|
| Success Rate | 75% | 92% | +17% |
| Constraint Violations | 8 | 2 | -6 |
| Avg. Completion Time | 45s | 38s | -7s |

**Conclusion**: Skills improve outcomes (+17% success rate)
```

### 3. 回帰検出

```bash
$ /skill-eval --regression CONS-NEW

## Regression Test for CONS-NEW

| Golden Task | Before | After | Impact |
|-------------|--------|-------|--------|
| GT-001 | PASS | PASS | None |
| GT-002 | PASS | FAIL | REGRESSION |
| GT-003 | PASS | PASS | None |

**Warning**: CONS-NEW causes regression in GT-002
**Recommendation**: Do not promote to Accepted
```

---

## 評価基準タイプ

| Type | 説明 | 例 |
|------|------|---|
| `contains` | パターンが含まれる | `"Hono"` in file |
| `not_contains` | パターンが含まれない | No `"express"` |
| `matches` | 正規表現マッチ | `/app\.(get|post)\(/` |
| `file_exists` | ファイルが存在 | `src/routes/user.ts` |
| `constraint_respected` | 制約違反なし | CONS-001 not triggered |
| `custom` | カスタムスクリプト | `scripts/validate.sh` |

---

## Baseline Mode

既存コードベースに対する「新規違反のみ」モード。

### 問題

既存プロジェクトに制約を適用すると、過去の違反がすべて検出される。
→ 「Everything fails forever」状態。

### 解決策

```yaml
# .claude/skill-config.yaml
evaluation:
  baseline_mode: true
  baseline_date: 2025-01-03  # この日以降の変更のみ評価
  existing_violations: debt   # 既存違反は技術的負債として記録
```

### 動作

```bash
$ /skill-for-review --staged

## Review Results

### New Violations (Block)
- CONS-001: src/routes/new.ts:15 - Uses Express

### Existing Violations (Debt - Info only)
- CONS-001: src/routes/legacy.ts:42 - Uses Express (pre-baseline)
- CONS-001: src/routes/old.ts:18 - Uses Express (pre-baseline)

**Status**: BLOCKED (1 new violation)
```

---

## Golden Task 作成

### 自動抽出

```bash
$ /skill-extract-golden

## Extracting Golden Tasks from History

Analyzing 50 recent sessions...

### Candidates

| Session | Quality | Description | Action |
|---------|---------|-------------|--------|
| abc123 | 0.92 | API endpoint creation | Extract? |
| def456 | 0.85 | Security review pattern | Extract? |
| ghi789 | 0.78 | Refactoring workflow | Extract? |

Select tasks to extract (e.g., "1, 2" or "all"):
```

### 手動作成

```yaml
# テンプレートから作成
$ /skill-create-golden

name: My Custom Golden Task
description: >
  Describe what this task tests

input:
  prompt: "The user request that triggers this scenario"
  files:
    - path: example.ts
      content: "// starting state"

expected:
  # Define success criteria
```

---

## 昇格ゲート

Golden Task は Maturity 昇格の必須条件。

### Draft → Accepted

```yaml
promotion_requirements:
  eval_pass_rate: 0.8        # 80%以上のGolden Task通過
  regression_allowed: false  # 回帰は許可しない
  min_golden_tasks: 3        # 最低3タスクで検証
```

### Accepted → Canonical

```yaml
promotion_requirements:
  eval_pass_rate: 0.95       # 95%以上
  regression_allowed: false
  min_golden_tasks: 5
  stability_period: 30d      # 30日間安定
```

---

## CI 統合

```yaml
# .github/workflows/skill-eval.yml
name: Skill Evaluation

on: [push, pull_request]

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Golden Tasks
        run: |
          skill-eval --ci --baseline

      - name: Check Regressions
        run: |
          skill-eval --regression --fail-on-regression
```

---

## 評価レポート形式

```markdown
# Skill Evaluation Report

**Date**: 2025-01-03
**Commit**: a1b2c3d

## Summary

| Metric | Value |
|--------|-------|
| Golden Tasks | 10 |
| Pass Rate | 90% |
| Regressions | 0 |
| New Violations | 2 |

## Constraint Performance

| Constraint | Tasks Tested | Pass Rate | Trend |
|------------|--------------|-----------|-------|
| CONS-001 | 8 | 100% | Stable |
| CONS-002 | 6 | 83% | Improving |
| CONS-003 | 4 | 75% | Declining ⚠️ |

## Recommendations

- CONS-003: Consider review (declining pass rate)
- CONS-NEW: Ready for promotion (100% pass, no regression)
```
