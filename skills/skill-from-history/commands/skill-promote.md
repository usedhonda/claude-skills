---
description: Promote constraints and skills through maturity levels. Use "/skill-promote", "/skill-promote CONS-XXX", or "/skill-promote --all-eligible".
---

# Skill Promote Command

制約とスキルの成熟度を昇格させる。

## Usage

```bash
/skill-promote                    # 昇格可能な候補を表示
/skill-promote CONS-XXX           # 特定制約を昇格
/skill-promote --all-eligible     # 全対象を一括昇格
/skill-promote --diff CONS-XXX    # 差分プレビュー
```

## Proposal-Commit Workflow

> "Observation → Drafting → Promotion → Refining" (Gemini)

### 1. Observe（観察）

履歴からパターンを検出。自動的に Draft として生成。

```bash
/gen-all  # Draft 状態で生成
```

### 2. Draft（下書き）

```
.claude/skills/[name]/
└── SKILL.md  (maturity: draft)
```

Draft 状態:
- 強制力: Warning のみ
- CI: 通知のみ、ブロックなし
- TTL: 30日（期限切れで自動削除提案）

### 3. Promote（昇格）

```bash
/skill-promote CONS-001
```

昇格チェックリスト:
- [ ] Eval Pass (>= 80%)
- [ ] Staleness Check (>= 0.5)
- [ ] No Regression
- [ ] Rationale 記録済み

### 4. Refine（改善）

既存スキルの更新・マージ・廃止。

```bash
/skill-refine CONS-001 --merge CONS-002
/skill-refine CONS-001 --deprecate
```

## 昇格条件

### Draft → Accepted

| 条件 | 要件 |
|------|------|
| Eval Pass Rate | >= 80% |
| Golden Tasks | >= 3 tasks tested |
| Regression | None |
| Rationale | Required (can be "UNKNOWN") |

### Accepted → Canonical

| 条件 | 要件 |
|------|------|
| Eval Pass Rate | >= 95% |
| Golden Tasks | >= 5 tasks tested |
| Stability Period | 30 days without drift |
| Override Frequency | < 2/week |
| Rationale | Required (must be specific) |

## 昇格レポート

```markdown
## Promotion Candidates

### Ready for Promotion (Draft → Accepted)

| ID | Pattern | Eval | Staleness | Action |
|----|---------|------|-----------|--------|
| CONS-001 | Express→Hono | 92% | 0.85 | [Promote] |
| CONS-002 | var→const | 88% | 0.72 | [Promote] |

### Needs Review

| ID | Pattern | Issue | Recommendation |
|----|---------|-------|----------------|
| CONS-003 | inline CSS | Eval 75% | Add more golden tasks |
| CONS-004 | any type | Staleness 0.45 | Reaffirm or deprecate |

### Not Ready

| ID | Pattern | Blocker |
|----|---------|---------|
| CONS-005 | legacy API | Regression detected |
```

## Rationale 収集

昇格時に Rationale が未記録の場合、プロンプトで収集：

```
CONS-001 の昇格準備ができています。

Pattern: Express.js router → Hono framework
Evidence: [E1][E2][E3]

このパターンを採用した理由を選択または入力してください:

1. Performance（パフォーマンス改善）
2. Security（セキュリティ強化）
3. Maintainability（保守性向上）
4. Team Decision（チーム決定）
5. Other（その他）: ___________

選択:
```

## 一括昇格

```bash
/skill-promote --all-eligible
```

対話的確認:

```
## Eligible for Promotion

| ID | Pattern | From | To |
|----|---------|------|-----|
| CONS-001 | Express→Hono | Draft | Accepted |
| CONS-002 | var→const | Draft | Accepted |
| SKILL-001 | api-patterns | Accepted | Canonical |

Promote all? (y/n/select):
```

## 差分プレビュー

```bash
/skill-promote --diff CONS-001
```

```diff
--- CONS-001 (Draft)
+++ CONS-001 (Accepted)

 maturity: draft
+maturity: accepted
+accepted_at: 2025-01-03
+rationale: "Team decision to migrate for better performance"

 ttl:
-  review_by: 2025-02-03
+  review_by: 2025-04-03
```

## Override 履歴の影響

頻繁な Override は昇格に影響:

| Override 頻度 | 影響 |
|--------------|------|
| 0-1/week | 昇格 OK |
| 2-3/week | レビュー推奨 |
| 4+/week | 昇格ブロック（制約見直し必要） |

## 関連ドキュメント

- [lifecycle.md](../references/lifecycle.md) - 成熟度モデル
- [golden-tasks.md](../references/golden-tasks.md) - 評価基準
- [evaluation.md](../references/evaluation.md) - 評価詳細
