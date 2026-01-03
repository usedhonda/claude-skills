---
description: Run Golden Tasks to evaluate constraints and skills. Use "/skill-eval", "/skill-eval --baseline", or "/skill-eval --regression CONS-XXX".
---

# Skill Evaluation Command

制約とスキルの有効性を Golden Tasks で評価する。

## Usage

```bash
/skill-eval                      # 全 Golden Tasks 実行
/skill-eval --baseline           # ベースライン比較
/skill-eval --regression CONS-XXX  # 特定制約の回帰テスト
/skill-eval --ci                 # CI モード（終了コード返却）
```

## Execution Flow

### 1. Golden Tasks 読み込み

```
.claude/golden-tasks/ から全タスク読み込み
├── index.yaml
├── GT-001.yaml
├── GT-002.yaml
└── ...
```

### 2. 各タスク実行

```
For each Golden Task:
  1. 入力コンテキストをセットアップ
  2. 制約を適用してシミュレート
  3. 期待結果と比較
  4. Pass/Fail 判定
```

### 3. 結果レポート

```markdown
## Golden Task Results

| Task | Status | Duration | Notes |
|------|--------|----------|-------|
| GT-001 | PASS | 2.3s | All criteria met |
| GT-002 | FAIL | 1.8s | CONS-003 violated |
| GT-003 | SKIP | - | Prerequisite not met |

## Summary
- Total: 3
- Passed: 1 (33%)
- Failed: 1
- Skipped: 1
```

## Options

### --baseline

制約 with/without の比較レポートを生成。

```markdown
## Baseline Comparison

| Metric | Without Skills | With Skills | Delta |
|--------|----------------|-------------|-------|
| Success Rate | 75% | 92% | +17% |
| Violations | 8 | 2 | -6 |

**Conclusion**: Skills improve outcomes
```

### --regression CONS-XXX

特定の制約が既存タスクを壊さないか検証。

```markdown
## Regression Test: CONS-NEW

| Task | Before | After | Impact |
|------|--------|-------|--------|
| GT-001 | PASS | PASS | None |
| GT-002 | PASS | FAIL | REGRESSION |

**Warning**: CONS-NEW causes regression
**Recommendation**: Do not promote
```

### --ci

CI 用の終了コード：
- 0: 全て Pass
- 1: 1つ以上 Fail
- 2: 回帰検出

## 昇格ゲート連携

`/skill-eval` の結果は制約の昇格に影響：

| 昇格 | 要件 |
|------|------|
| Draft → Accepted | Pass Rate >= 80%, No Regression |
| Accepted → Canonical | Pass Rate >= 95%, 30日安定 |

## Golden Task 作成

```bash
/skill-extract-golden  # 履歴から自動抽出
/skill-create-golden   # 手動作成
```

## 関連ドキュメント

- [golden-tasks.md](../references/golden-tasks.md) - 詳細仕様
- [lifecycle.md](../references/lifecycle.md) - 昇格ルール
