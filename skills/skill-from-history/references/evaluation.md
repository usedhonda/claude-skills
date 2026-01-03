# Evaluation System

制約とスキルの評価システム詳細仕様。

---

## アーキテクチャ

```
┌─────────────────┐
│  Golden Tasks   │
│  (.claude/      │
│  golden-tasks/) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│  Eval Runner    │────►│  Constraint     │
│                 │     │  Matcher        │
└────────┬────────┘     └─────────────────┘
         │
         ▼
┌─────────────────┐
│  Result Report  │
│  (JSON/MD)      │
└─────────────────┘
```

---

## Golden Task 詳細

### タスク構造

```yaml
id: GT-001
name: API Endpoint Creation
version: 1.0
tags: [api, hono, endpoint]

# 前提条件
prerequisites:
  files_exist:
    - src/routes/index.ts
  constraints_active:
    - CONS-001

# 入力
input:
  prompt: "Create a new API endpoint for user profile"
  context:
    project_type: hono-api
    existing_patterns:
      - "src/routes/*.ts"

# シミュレーション設定
simulation:
  mode: dry-run  # dry-run | actual
  timeout: 30s
  tools_allowed:
    - Read
    - Write
    - Glob

# 期待結果
expected:
  files:
    created:
      - path: src/routes/user.ts
        contains:
          - pattern: "import { Hono }"
            required: true
          - pattern: "app.get('/profile'"
            required: true
        not_contains:
          - pattern: "express"
            severity: critical
          - pattern: "Router()"
            severity: warning

    modified:
      - path: src/routes/index.ts
        contains:
          - pattern: "import.*user"

  constraints:
    respected: [CONS-001, CONS-002]
    violated: []

# 評価基準（重み付き）
criteria:
  - id: C1
    name: Uses Hono framework
    type: contains
    target: src/routes/user.ts
    pattern: "Hono"
    weight: 1.0
    required: true

  - id: C2
    name: No Express usage
    type: not_contains
    target: src/routes/user.ts
    pattern: "express|Router"
    weight: 0.8
    required: true

  - id: C3
    name: Route naming convention
    type: matches
    target: src/routes/user.ts
    pattern: "app\\.(get|post|put|delete)\\('/"
    weight: 0.5
    required: false
```

### 評価基準タイプ

| Type | 説明 | パラメータ |
|------|------|-----------|
| `contains` | パターンが含まれる | `pattern`, `target` |
| `not_contains` | パターンが含まれない | `pattern`, `target` |
| `matches` | 正規表現マッチ | `pattern`, `target` |
| `file_exists` | ファイル存在確認 | `path` |
| `file_not_exists` | ファイル不在確認 | `path` |
| `constraint_respected` | 制約違反なし | `constraint_id` |
| `constraint_violated` | 制約違反あり（期待） | `constraint_id` |
| `output_contains` | 出力にパターン | `pattern` |
| `custom` | カスタムスクリプト | `script`, `args` |

---

## 評価プロセス

### 1. 前提条件チェック

```python
def check_prerequisites(task):
    for file in task.prerequisites.files_exist:
        if not exists(file):
            return Skip(f"File not found: {file}")

    for constraint in task.prerequisites.constraints_active:
        if not is_active(constraint):
            return Skip(f"Constraint inactive: {constraint}")

    return Continue()
```

### 2. シミュレーション実行

```python
def run_simulation(task):
    context = setup_context(task.input)

    # Dry-run モード: 実際のファイル変更なし
    if task.simulation.mode == "dry-run":
        result = simulate_with_constraints(
            prompt=task.input.prompt,
            context=context,
            constraints=get_active_constraints(),
            timeout=task.simulation.timeout
        )
    else:
        result = execute_actual(task)

    return result
```

### 3. 結果検証

```python
def verify_results(task, actual):
    scores = []

    for criterion in task.criteria:
        score = evaluate_criterion(criterion, actual)
        scores.append({
            "id": criterion.id,
            "name": criterion.name,
            "passed": score >= criterion.threshold,
            "score": score,
            "weight": criterion.weight,
            "required": criterion.required
        })

    # 必須基準が全て Pass なら成功
    required_passed = all(s["passed"] for s in scores if s["required"])

    # 重み付きスコア計算
    weighted_score = sum(s["score"] * s["weight"] for s in scores) / sum(s["weight"] for s in scores)

    return EvalResult(
        passed=required_passed,
        score=weighted_score,
        criteria=scores
    )
```

---

## ベースライン比較

### 概念

同じ Golden Tasks を「制約なし」と「制約あり」で実行し、改善度を測定。

### 実行フロー

```python
def baseline_comparison(tasks):
    # 1. 制約なしで実行
    without_results = run_tasks(tasks, constraints_enabled=False)

    # 2. 制約ありで実行
    with_results = run_tasks(tasks, constraints_enabled=True)

    # 3. 比較
    comparison = {
        "without": {
            "success_rate": calc_success_rate(without_results),
            "violations": count_violations(without_results)
        },
        "with": {
            "success_rate": calc_success_rate(with_results),
            "violations": count_violations(with_results)
        }
    }

    comparison["improvement"] = (
        comparison["with"]["success_rate"] -
        comparison["without"]["success_rate"]
    )

    return comparison
```

### レポート形式

```markdown
## Baseline Comparison Report

**Date**: 2025-01-03
**Tasks**: 10

### Summary

| Metric | Without | With | Delta |
|--------|---------|------|-------|
| Success Rate | 70% | 90% | **+20%** |
| Violations | 12 | 3 | **-9** |
| Avg Duration | 45s | 42s | -3s |

### Per-Constraint Impact

| Constraint | Tasks Affected | Improvement |
|------------|----------------|-------------|
| CONS-001 | 5 | +25% |
| CONS-002 | 3 | +15% |
| CONS-003 | 2 | -5% ⚠️ |

### Recommendations

- CONS-001: High impact, keep
- CONS-002: Moderate impact, keep
- CONS-003: Negative impact, review
```

---

## 回帰テスト

### 目的

新しい制約が既存の成功タスクを壊さないことを確認。

### 実行フロー

```python
def regression_test(new_constraint):
    # 1. 現在の制約セットで実行
    baseline = run_tasks(all_tasks, constraints=current_constraints)

    # 2. 新制約を追加して実行
    with_new = run_tasks(all_tasks, constraints=current_constraints + [new_constraint])

    # 3. 回帰検出
    regressions = []
    for task in all_tasks:
        if baseline[task].passed and not with_new[task].passed:
            regressions.append({
                "task": task,
                "before": "PASS",
                "after": "FAIL",
                "cause": analyze_cause(task, new_constraint)
            })

    return RegressionResult(
        regressions=regressions,
        has_regression=len(regressions) > 0
    )
```

### 回帰検出時のアクション

| 回帰数 | 推奨アクション |
|--------|---------------|
| 0 | 昇格 OK |
| 1-2 | レビュー推奨 |
| 3+ | 昇格ブロック |

---

## Baseline Mode（既存違反の扱い）

### 問題

既存プロジェクトに制約を適用すると、過去のコードがすべて違反として検出される。

### 解決策

```yaml
# .claude/skill-config.yaml
evaluation:
  baseline_mode: true
  baseline_date: 2025-01-03
  existing_violations:
    treatment: debt  # debt | ignore | warn
    tracking: true
```

### 動作

```python
def evaluate_with_baseline(file, constraint, baseline_date):
    violation = check_violation(file, constraint)

    if violation:
        file_modified = get_last_modified(file)

        if file_modified < baseline_date:
            # 既存違反（baseline前）
            return Debt(violation)
        else:
            # 新規違反（baseline後）
            return Violation(violation)

    return Pass()
```

### レポート分類

```markdown
## Review Results

### New Violations (Block)
- src/routes/new.ts:15 - CONS-001

### Existing Violations (Debt)
- src/routes/legacy.ts:42 - CONS-001 (pre-baseline)
- src/routes/old.ts:18 - CONS-001 (pre-baseline)

### Summary
- New: 1 (blocking)
- Debt: 2 (info only)
```

---

## CI 統合

### GitHub Actions

```yaml
name: Skill Evaluation

on:
  push:
    branches: [main]
  pull_request:

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Golden Tasks
        run: skill-eval --ci
        continue-on-error: true

      - name: Check Regressions
        run: skill-eval --regression --fail-on-regression

      - name: Upload Report
        uses: actions/upload-artifact@v4
        with:
          name: eval-report
          path: .claude/eval-results/
```

### 終了コード

| Code | 意味 |
|------|------|
| 0 | 全て Pass |
| 1 | 1つ以上 Fail |
| 2 | 回帰検出 |
| 3 | 設定エラー |

---

## 結果保存

### ファイル形式

```
.claude/eval-results/
├── latest.json
├── 2025-01-03T12:00:00.json
└── history.json
```

### JSON 形式

```json
{
  "timestamp": "2025-01-03T12:00:00Z",
  "commit": "a1b2c3d",
  "summary": {
    "total": 10,
    "passed": 9,
    "failed": 1,
    "skipped": 0,
    "pass_rate": 0.9
  },
  "tasks": [
    {
      "id": "GT-001",
      "status": "pass",
      "duration_ms": 2300,
      "criteria": [...]
    }
  ],
  "constraints": {
    "CONS-001": { "tested": 8, "pass_rate": 1.0 },
    "CONS-002": { "tested": 6, "pass_rate": 0.83 }
  }
}
```
