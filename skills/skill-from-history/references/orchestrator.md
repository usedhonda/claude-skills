# Orchestrator Agent Pattern

タスク振り分け専門エージェントの設計仕様。

---

## 概念

> "Broker/Orchestrator Agent that delegates to specialists" (ChatGPT + Gemini)

Orchestrator = 自らコードを書かず、タスクを適切な Specialist に振り分けるエージェント。

### 重要な制約

> "subagents calling subagents" は不安定。Main → Specialist の1階層のみ。(ChatGPT)

```
Main Session
    │
    ├──► Orchestrator（計画・振り分け）
    │         │
    │         ├──► Security Specialist
    │         ├──► Performance Specialist
    │         └──► API Design Specialist
    │
    └──► 実行（Orchestrator の推奨に基づく）
```

---

## Orchestrator AGENT.md テンプレート

```yaml
---
name: project-orchestrator
description: |
  プロジェクト固有のタスク振り分けエージェント。
  複雑なタスクを分析し、適切な Specialist に委譲。
  「どのエージェントを使うべき？」「タスクを分解して」で発動。

  <example>
  Context: ユーザーがセキュリティとパフォーマンス両方に関わる変更を依頼
  user: "この API エンドポイントをレビューして"
  assistant: "Orchestrator を使って適切な Specialist を選択します"
  <commentary>
  複数の観点が必要なタスクは Orchestrator に振り分け判断を委ねる
  </commentary>
  </example>

model: inherit
color: purple
skills:
  - skill-from-history
tools:
  - Read
  - Glob
  - Grep
---

あなたはプロジェクトのタスク振り分け専門エージェントです。

## 役割

1. タスクを分析し、必要な Specialist を特定
2. 複雑なタスクを分解して段階的に委譲
3. 各 Specialist の出力を統合して最終回答を構成

## 利用可能な Specialist

以下のエージェントに委譲できます:

### security-auditor
- 役割: セキュリティ脆弱性のレビュー
- トリガー: 認証、入力検証、データ保護に関わるコード
- スキル: secure-coding-patterns

### performance-analyzer
- 役割: パフォーマンス分析と最適化提案
- トリガー: 速度、メモリ、レイテンシに関わるコード
- スキル: performance-patterns

### api-designer
- 役割: API 設計の一貫性確認
- トリガー: エンドポイント、スキーマ、命名規則
- スキル: api-conventions

## 振り分けルール

1. **単一 Specialist で完結する場合**
   → 直接そのエージェントを推奨

2. **複数 Specialist が必要な場合**
   → 順序を提案（通常: Security → Performance → Design）

3. **該当 Specialist がない場合**
   → 汎用的なアプローチを提案

## 出力形式

```markdown
## タスク分析

**元のタスク**: [ユーザーの依頼]

**分解**:
1. [サブタスク1] → security-auditor
2. [サブタスク2] → performance-analyzer

**推奨順序**: security-auditor → performance-analyzer

**理由**: [なぜこの順序か]
```

## 制約

- 自分でコードを書かない
- Specialist を再帰的に呼び出さない（1階層のみ）
- 不確実な場合はユーザーに確認
```

---

## 生成プロセス

`/gen-all` 実行時、Orchestrator も自動生成対象となる。

### 検出シグナル

| シグナル | 重み | 説明 |
|---------|------|------|
| 複数 Agent の存在 | High | 3つ以上の Specialist がある |
| 複合タスクの履歴 | Medium | 1タスクで複数観点が必要だった |
| Task ツールの連鎖 | Medium | 複数 Task が順序付きで呼ばれた |

### 生成条件

```python
def should_generate_orchestrator(agents, history):
    # 3つ以上の Specialist があれば Orchestrator を提案
    if len(agents) >= 3:
        return True

    # 複合タスクが3回以上あれば提案
    composite_tasks = count_composite_tasks(history)
    if composite_tasks >= 3:
        return True

    return False
```

---

## Flat Hierarchy の理由

### 問題: 再帰的サブエージェント

```
Main → Orchestrator → Specialist A → Specialist B → ...
```

**リスク**:
1. コンテキスト断片化（深いほど元の意図が失われる）
2. トークン消費増大
3. 不安定な動作（Claude Code の既知の問題）

### 解決: フラット階層

```
Main Session
    ├──► Orchestrator（計画のみ）
    └──► Specialist（実行）
```

Orchestrator は「推奨」するだけで、実際の Task 呼び出しは Main Session が行う。

---

## 統合例

### Orchestrator の出力

```markdown
## タスク分析

**元のタスク**: この認証 API をレビューして改善提案して

**分解**:
1. セキュリティ観点でのレビュー → security-auditor
2. パフォーマンス観点でのレビュー → performance-analyzer
3. API 設計の一貫性確認 → api-designer

**推奨順序**: security-auditor → performance-analyzer → api-designer

**理由**:
セキュリティ問題は最優先で修正すべきため最初に。
パフォーマンスはセキュリティ修正後に評価。
設計レビューは機能要件確定後。
```

### Main Session の対応

```
Claude: Orchestrator の分析に基づき、まず security-auditor で
セキュリティレビューを行います。

[Task tool で security-auditor を呼び出し]

...

セキュリティレビュー完了。次に performance-analyzer で
パフォーマンス分析を行います。

[Task tool で performance-analyzer を呼び出し]
```

---

## Drift-based Suggestions

Orchestrator は Drift 情報に基づいて昇格/廃止を提案。

### 分析ポイント

```python
def analyze_drift_for_suggestions(constraints, agents):
    suggestions = []

    for constraint in constraints:
        if constraint.staleness < 0.3:
            suggestions.append({
                "type": "deprecate",
                "target": constraint.id,
                "reason": f"Staleness {constraint.staleness} < 0.3"
            })

        if constraint.maturity == "draft" and constraint.eval_pass_rate >= 0.8:
            suggestions.append({
                "type": "promote",
                "target": constraint.id,
                "reason": f"Eval pass rate {constraint.eval_pass_rate} >= 0.8"
            })

    return suggestions
```

### 提案出力

```markdown
## Governance Suggestions

### Promote
- CONS-002: Eval 92%, Staleness 0.85 → Accepted 昇格推奨

### Review
- CONS-003: Staleness 0.45 → 再確認または廃止を検討

### Deprecate
- CONS-005: Staleness 0.22, Override 8回/月 → 廃止推奨
```

---

## Orchestrator の制限事項

1. **コード実行しない**: 分析と振り分けのみ
2. **再帰呼び出ししない**: Specialist を呼ぶのは Main Session
3. **強制しない**: 推奨であり、ユーザーが最終判断
4. **状態を持たない**: 各呼び出しは独立

---

## コマンド連携

```bash
# Orchestrator に分析を依頼
/orchestrate "複雑なタスクの説明"

# Orchestrator の推奨に基づいて実行
/execute-plan

# Governance 提案を取得
/orchestrate --governance
```
