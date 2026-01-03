# Semantic Conflict Detection

Algorithms for detecting and resolving contradicting rules in constraints and patterns.

## Conflict Types

### 1. Direct Contradiction

Same subject with opposite polarity.

**Example**:
- Existing: "Use Express.js for API routes"
- New: "Don't use Express.js for API routes"

**Detection**:
```
subject_similarity > 0.8 AND polarity_opposite = true
```

### 2. Temporal Conflict (Evolution)

Newer constraint supersedes older one.

**Example**:
- Old (2024-10): "Use dayjs for date operations"
- New (2024-12): "Use date-fns for date operations"

**Detection**:
```
subject_similarity > 0.8 AND same_action_type AND different_target
```

### 3. Scope Conflict

Same context with different recommended actions.

**Example**:
- Constraint A: "Use CSS modules for component styling"
- Constraint B: "Use Tailwind for component styling"

**Detection**:
```
context_overlap > 0.6 AND different_action
```

### 4. Conditional Conflict

Rules that apply to overlapping but not identical conditions.

**Example**:
- "Always use TypeScript"
- "Use JavaScript for config files"

**Detection**:
```
broader_scope_constraint conflicts with narrower_scope_exception
```

## Detection Algorithm

### Subject Extraction

Extract the core subject from constraint patterns:

```
Input: "Don't use Express.js router for API endpoints"
Output: {
  action: "use",
  polarity: "negative",
  subject: "Express.js router",
  context: "API endpoints"
}
```

### Similarity Calculation

```
subject_similarity = semantic_similarity(C1.subject, C2.subject)
context_overlap = jaccard_index(C1.context_keywords, C2.context_keywords)
polarity_match = (C1.polarity == C2.polarity)
```

### Conflict Classification

```python
def classify_conflict(new_constraint, existing_constraint):
    subject_sim = semantic_similarity(new.subject, existing.subject)
    context_sim = context_overlap(new.context, existing.context)

    if subject_sim < 0.5:
        return None  # No conflict

    if subject_sim > 0.8 and polarity_opposite(new, existing):
        return "direct_contradiction"

    if subject_sim > 0.8 and same_action_type(new, existing):
        if new.timestamp > existing.timestamp:
            return "temporal_evolution"
        else:
            return "temporal_regression"

    if context_sim > 0.6 and different_action(new, existing):
        return "scope_conflict"

    return "potential_conflict"  # Requires manual review
```

## Resolution Workflow

### Presentation Format

```markdown
## Conflict Detected

**New constraint**: Use date-fns for date operations [E15]
**Conflicts with**: Use dayjs for date operations [E3]

| Aspect | Existing (E3) | New (E15) |
|--------|---------------|-----------|
| Date | 2024-10-15 | 2024-12-20 |
| Confidence | 72% | 85% |
| Frequency | 2 | 4 |
| Evidence | [E3] | [E15][E16] |

**Conflict type**: Temporal Evolution

**Resolution options**:
1. `supersede` - Replace old with new (recommended for evolution)
2. `scope` - Both valid in different contexts
3. `keep-old` - Reject new constraint
4. `merge` - Combine into conditional constraint
```

### Resolution Options

| Option | Action | When to Use |
|--------|--------|-------------|
| `supersede` | Mark old as superseded, link to new | Technology migration |
| `scope` | Add context conditions to both | Different use cases |
| `keep-old` | Reject new constraint | New constraint is incorrect |
| `merge` | Create conditional rule | Both valid with conditions |

### Supersession Handling

When `supersede` is chosen:

1. Update old constraint:
```json
{
  "id": "constraint-dayjs",
  "status": "superseded",
  "superseded_by": "constraint-datefns",
  "superseded_at": "2024-12-20T15:30:00Z"
}
```

2. Update new constraint:
```json
{
  "id": "constraint-datefns",
  "supersedes": "constraint-dayjs",
  "supersession_note": "Team migrated to date-fns for tree-shaking"
}
```

3. Auto-generate ADR (if ADR-lite enabled)

### Scope Resolution

When `scope` is chosen:

```json
{
  "id": "constraint-dayjs",
  "scope": "legacy code maintenance",
  "scope_note": "Use dayjs only for existing code"
}

{
  "id": "constraint-datefns",
  "scope": "new development",
  "scope_note": "Use date-fns for all new code"
}
```

## Conflict Storage

### Conflict Record

```json
{
  "conflict_id": "conflict-abc123",
  "type": "temporal_evolution",
  "detected_at": "2024-12-20T15:00:00Z",
  "constraints": {
    "existing": "constraint-dayjs",
    "new": "constraint-datefns"
  },
  "resolution": {
    "status": "resolved",
    "action": "supersede",
    "resolved_at": "2024-12-20T15:30:00Z",
    "resolved_by": "user",
    "note": "Team migrated to date-fns"
  }
}
```

### Unresolved Conflict Handling

Unresolved conflicts:
- Block skill generation if Critical severity involved
- Show warning in skill-for-review output
- Trigger CONS-005 lint error

## Integration Points

### With Pattern Detection

During Step 4 (Extract Pattern Candidates):
- Check new patterns against existing patterns
- Flag potential duplicates or conflicts

### With Anti-pattern Detection

During Step 4.5 (Extract Anti-patterns):
- Check if anti-pattern contradicts existing positive pattern
- Priority: Anti-pattern wins (safety first)

### With Review Gate

During Step 7.5 (Review Gate):
- Show conflicts alongside Pending Critical constraints
- Require resolution before Critical promotion

### With ADR-lite

On supersession resolution:
- Auto-propose ADR generation
- Include both old and new evidence

## Performance Considerations

### Caching

- Cache extracted subjects and contexts
- Incremental conflict checking (only new vs existing)

### Thresholds

| Parameter | Default | Description |
|-----------|---------|-------------|
| subject_similarity_threshold | 0.8 | Minimum for conflict detection |
| context_overlap_threshold | 0.6 | Minimum for scope conflict |
| max_conflicts_per_batch | 10 | Stop after finding N conflicts |

## Temporal Weighting

Automatic resolution preference based on evidence recency.

### Default Behavior

When conflicts are detected, newer evidence is preferred by default:

| Scenario | Resolution |
|----------|------------|
| Same confidence | Prefer newer evidence |
| Confidence differs by >10% | Prefer higher confidence |
| Both similar | Prompt user for decision |

### Recency Weight Calculation

Evidence weight decreases with age:

```
recency_weight = 1.0 - (days_old / 365) * 0.5
```

| Age | Weight |
|-----|--------|
| Today | 1.0 |
| 3 months | 0.875 |
| 6 months | 0.75 |
| 1 year | 0.5 |

### Evidence Timestamp Format

```json
{
  "evidence": [
    {
      "id": "E1",
      "timestamp": "2024-10-15",
      "raw_confidence": 0.85,
      "recency_weight": 0.7,
      "weighted_confidence": 0.595
    },
    {
      "id": "E2",
      "timestamp": "2024-12-20",
      "raw_confidence": 0.80,
      "recency_weight": 1.0,
      "weighted_confidence": 0.80
    }
  ]
}
```

### Weighted Conflict Resolution

```python
def resolve_temporal_conflict(existing, new):
    existing_score = existing.confidence * existing.recency_weight
    new_score = new.confidence * new.recency_weight

    diff = abs(existing_score - new_score)

    if diff < 0.1:  # Within 10%
        return "prompt_user"
    elif new_score > existing_score:
        return "supersede"
    else:
        return "keep_old"
```

### Override Options

Users can explicitly override temporal weighting:

```bash
# Force keep old rule despite newer evidence
resolution: "keep-old" --reason "Legacy compatibility required"

# Force supersede despite lower confidence
resolution: "supersede" --reason "Team decision to migrate"
```

Override reasons are recorded in conflict resolution log.

## Skill Update Detection

When generating new skills, check for updates to existing skills.

### Update Detection Flow

```
新規パターン vs 既存スキル
    │
    ├─ >= 80%（重複）
    │   ├─ 新パターンあり → 更新提案
    │   └─ 新パターンなし → スキップ
    │
    ├─ 50-79%（類似）→ 拡張提案（別スキルとして）
    │
    └─ < 50%（新規）→ 新規生成
```

### 新パターン検出項目

| 項目 | 検出方法 | 更新箇所 |
|------|---------|---------|
| 新エビデンス | 新しいセッション/コミット | Evidence Index |
| 新制約 | 新しい rejection パターン | Constraints セクション |
| 新手順 | 新しいワークフローステップ | Core Pattern セクション |
| 新ベストプラクティス | 繰り返しの推奨パターン | Best Practices セクション |
| 新エージェント参照 | 推奨エージェントの検出 | agents: フィールド |

### 更新提案表示

```markdown
## Existing Skill Updates

| Skill | Match | Updates | Details |
|-------|-------|---------|---------|
| multi-ai-review | 88% | +2 evidence, +1 step | New synthesis step |
| secure-coding | 75% | +3 constraints | New injection patterns |
| api-design | 92% | (none) | No new patterns |

[S1] Update multi-ai-review? (y/n/diff):
[S2] Update secure-coding? (y/n/diff):
```

### 更新マージ処理

```python
def merge_skill_updates(existing: Skill, new_patterns: Patterns) -> Skill:
    # 1. Evidence Index に追加
    for evidence in new_patterns.evidence:
        if evidence not in existing.evidence:
            existing.evidence.append(evidence)

    # 2. Constraints に追加（重複チェック付き）
    for constraint in new_patterns.constraints:
        if not has_similar_constraint(existing.constraints, constraint):
            existing.constraints.append(constraint)

    # 3. Core Pattern にステップ追加
    for step in new_patterns.steps:
        if step.is_new_phase:
            existing.core_pattern.insert_step(step)

    # 4. agents: フィールドに追加
    for agent in new_patterns.recommended_agents:
        if agent not in existing.agents:
            existing.agents.append(agent)

    # 5. updated_at 記録
    existing.updated_at = now()

    return existing
```

### 差分表示（diff オプション）

```diff
## Core Pattern

### Step 1: Prepare Context
  [unchanged]

### Step 2: Ask ChatGPT
  [unchanged]

+ ### Step 2.5: Cross-validate
+ Compare ChatGPT response with project constraints.

### Step 3: Ask Gemini
  [unchanged]

## Constraints

| Pattern | Instead | Reason | Evidence |
|---------|---------|--------|----------|
  Single AI review | Multi-AI approach | Coverage | [E1] |
+ Skip synthesis | Always synthesize | Quality | [E12][E13] |
```

### 更新履歴

更新されたスキルには履歴を記録：

```yaml
---
name: multi-ai-review
description: ...
updated_at: 2025-01-03
update_history:
  - version: 1.1
    date: 2025-01-03
    changes: "+1 step (Cross-validate), +2 evidence"
  - version: 1.0
    date: 2024-12-15
    changes: "Initial generation"
---
```
