# Lifecycle Management

Constraint、Agent、Skill のライフサイクル管理仕様。

---

## Constraint Maturity

制約の成熟度モデル。

### 状態遷移

```
Draft ──[eval pass]──► Accepted ──[stability]──► Canonical
                           │                         │
                           ▼                         ▼
                      Deprecated ◄────────────── Deprecated
```

### 成熟度レベル

| Level | 説明 | 強制力 | 昇格条件 |
|-------|------|--------|----------|
| **Draft** | 提案段階 | Warning のみ | 自動生成時のデフォルト |
| **Accepted** | 承認済み | Warning + CI Warning | Eval Pass + User Approval |
| **Canonical** | 標準規則 | Error + CI Block | Stability期間 + No Drift |
| **Deprecated** | 非推奨 | Info のみ | Manual or Drift Detection |

### YAML表現

```yaml
constraints:
  - id: CONS-001
    pattern: "Express.js router"
    instead: "Hono framework"
    maturity: accepted
    created_at: 2024-12-15
    accepted_at: 2025-01-03
    review_by: 2025-04-03  # TTL: 90日後
    evidence: [E1, E2, E3]
    rationale: "Team decision to migrate"  # 必須
```

### 昇格の絶対ルール

Canonical への昇格には以下すべてが必要:

1. **Eval Pass**: Golden Task suite を通過（または明示的 override）
2. **Staleness Check**: 直近30日以内に reaffirmation または使用
3. **Rationale**: 理由の記録（`UNKNOWN` でも acknowledgement 必須）

---

## Epochs Model

プロジェクトの時代区分モデル。

### 概念

Epoch = プロジェクトが異なるアーキテクチャ/方針で運用された期間。
古い Epoch のルールは現在の Epoch では適用されない（または Warning に降格）。

### 定義ファイル

**場所**: `.claude/epochs.yaml` または `references/epochs.yaml`

```yaml
epochs:
  - id: E1
    name: monolith-era
    status: deprecated
    started_at: 2023-01-01
    ended_at: 2024-10-01
    description: "Single Rails monolith"

  - id: E2
    name: migration-era
    status: active
    started_at: 2024-10-01
    description: "Gradual migration to microservices"
    constraints_mode: mixed  # E1 rules warn, E2 rules enforce

  - id: E3
    name: microservices-era
    status: planned
    description: "Full microservices architecture"

current_epoch: E2
```

### Epoch と Constraint の関係

```yaml
constraints:
  - id: CONS-001
    epoch_id: E2       # この Epoch でのみ有効
    supersedes: CONS-OLD-001  # E1 のルールを置き換え
```

### Epoch フィルタリング

```bash
# 現在の Epoch のみ適用
skill-for-review --epoch current

# 移行期：両方を警告表示
skill-for-review --epoch E1,E2

# 特定 Epoch の履歴を再構築
skill-for-review --epoch E1
```

---

## Staleness Scoring

ルールの「鮮度」を定量化。

### スコア計算

```python
def calculate_staleness(constraint):
    # 基本減衰（365日で50%）
    days_old = (now() - constraint.last_activity).days
    base_score = 1.0 - (days_old / 365) * 0.5

    # 再確認ボーナス（+0.2）
    if constraint.reaffirmed_recently:
        base_score += 0.2

    # 違反ペナルティ（-0.1 per violation）
    base_score -= constraint.recent_violations * 0.1

    # オーバーライドペナルティ（-0.15 per override）
    base_score -= constraint.recent_overrides * 0.15

    return max(0, min(1.0, base_score))
```

### 活動タイプ

| Activity | 効果 | 説明 |
|----------|------|------|
| Reaffirmation | +0.2 | ユーザーが明示的に再確認 |
| Evidence Added | +0.1 | 新しいエビデンス追加 |
| Violation | -0.1 | ルール違反の検出 |
| Override | -0.15 | 例外的に無視された |
| Time Decay | -0.001/日 | 自然減衰 |

### Staleness 閾値

| Score | Status | Action |
|-------|--------|--------|
| >= 0.7 | Fresh | 正常 |
| 0.5-0.69 | Aging | Review 推奨 |
| 0.3-0.49 | Stale | Drift Alert 発火 |
| < 0.3 | Expired | 自動 Deprecation 提案 |

### Staleness YAML

```yaml
constraints:
  - id: CONS-001
    staleness:
      score: 0.72
      last_activity: 2025-01-02
      reaffirmations: 2
      violations: 1
      overrides: 0
      status: fresh
```

---

## Drift Detection

パターンの「ドリフト」（変化）を検出。

### ドリフトシグナル

| Signal | Detection | Threshold |
|--------|-----------|-----------|
| **Pattern Disappearance** | パターンが30日以上出現しない | Warning |
| **Violation Spike** | 違反が3倍以上に増加 | Alert |
| **Override Frequency** | 週に3回以上オーバーライド | Alert |
| **Evidence Contradiction** | 新エビデンスが既存と矛盾 | Critical |

### Drift Alert 形式

```markdown
## Drift Alert: CONS-001

**Pattern**: Express.js router → Hono
**Status**: Pattern disappeared for 45 days

**Evidence**:
- Last seen: 2024-11-15 (session:abc123)
- Recent violations: 0
- Recent overrides: 5

**Recommendation**:
- [ ] Deprecate (pattern no longer relevant)
- [ ] Reaffirm (still valid, just not used recently)
- [ ] Update (modify the constraint)
```

### Drift への対応

| Response | Action |
|----------|--------|
| `deprecate` | Maturity → Deprecated |
| `reaffirm` | Staleness score +0.3 |
| `update` | 新パターンとしてマージ |
| `ignore` | 30日間アラート抑制 |

---

## TTL (Time To Live)

ルールの有効期限管理。

### TTL 設定

```yaml
constraints:
  - id: CONS-001
    maturity: accepted
    ttl:
      review_by: 2025-04-03    # レビュー期限
      expires_at: 2025-07-03   # 自動 deprecation
      auto_renew: false        # 自動更新なし
```

### TTL ポリシー

| Maturity | Default TTL | Auto-Renew |
|----------|-------------|------------|
| Draft | 30 days | No |
| Accepted | 90 days | Optional |
| Canonical | 180 days | Yes (if Eval pass) |

### TTL 満了時の動作

1. `review_by` 到達 → Drift Alert 発火
2. `expires_at` 到達 → Maturity → Deprecated
3. Auto-Renew 有効 + Eval Pass → TTL 延長

---

## Override Protocol

制約を例外的に無視するための安全なエスケープハッチ。

### 概念

> "Guardrails, not handcuffs" (ChatGPT)

制約がイノベーションをブロックする場合、明示的な理由付きで override 可能。
ただし override は記録され、staleness スコアに影響する。

### Override 形式

#### コミットメッセージ

```
feat: add new API endpoint

ALLOW_CONSTRAINT: CONS-001
REASON: Legacy client compatibility required until Q2
```

#### PR / コード内アノテーション

```typescript
// ALLOW_CONSTRAINT: CONS-001
// REASON: Using Express for legacy endpoint, will migrate in ISSUE-123
app.use('/legacy', expressRouter);
```

### Override 記録

```yaml
overrides:
  - constraint_id: CONS-001
    commit: a1b2c3d
    date: 2025-01-03
    reason: "Legacy client compatibility"
    author: user@example.com
    issue_ref: ISSUE-123  # Optional: tracking issue
```

### Override ポリシー

| Maturity | Override 可否 | 影響 |
|----------|---------------|------|
| Draft | Always | No impact |
| Accepted | With reason | Staleness -0.15 |
| Canonical | With reason + issue | Staleness -0.15, Drift signal |
| Deprecated | N/A | Already inactive |

### Override 頻度監視

```
週3回以上の Override → Drift Alert
月10回以上 → 自動 Deprecation 提案
```

### Override の追跡

```bash
$ /skill-overrides

## Recent Overrides

| Constraint | Count (30d) | Last Reason | Action |
|------------|-------------|-------------|--------|
| CONS-001 | 5 | Legacy compat | Review |
| CONS-002 | 1 | Edge case | OK |
```

---

## Integration

### ライフサイクル状態の表示

```bash
$ /skill-status

## Constraint Status

| ID | Pattern | Maturity | Staleness | TTL | Action |
|----|---------|----------|-----------|-----|--------|
| CONS-001 | Express→Hono | Canonical | 0.82 | 45d | - |
| CONS-002 | var→const | Accepted | 0.45 | 12d | Review |
| CONS-003 | inline CSS | Draft | 0.31 | 5d | Drift Alert |
```

### CI 統合

```yaml
# .github/workflows/skill-check.yml
- name: Skill Governance Check
  run: |
    skill-for-review --staged --maturity-filter accepted,canonical
```

### SKILL.md への組み込み

生成されるスキルには自動的にライフサイクル情報が含まれる：

```yaml
---
name: api-patterns
lifecycle:
  maturity: accepted
  epoch: E2
  staleness: 0.75
  review_by: 2025-04-03
---
```
