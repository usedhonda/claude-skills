# ADR-lite: Lightweight Architecture Decision Records

Automatic generation of Architecture Decision Records for high-confidence patterns.

## Trigger Conditions

ADR generation is proposed when a pattern meets these thresholds:

| Metric | Threshold | Rationale |
|--------|-----------|-----------|
| Confidence | >= 90% | High certainty pattern |
| Evidence count | >= 5 | Well-documented decision |
| Frequency | >= 5 | Consistently applied |

Any one threshold being met triggers ADR proposal.

## ADR Format

Lightweight format optimized for developer consumption:

```markdown
# ADR-NNN: [Decision Title]

**Date**: YYYY-MM-DD
**Status**: Accepted | Superseded | Deprecated
**Confidence**: NN%

## Context

[Why this decision was needed - extracted from pattern context]

## Decision

[What was decided - the core pattern]

## Consequences

### Positive
- [Benefits observed from evidence]

### Negative
- [Trade-offs or limitations]

## Evidence

| Source | Date | Excerpt |
|--------|------|---------|
| [E1] session:abc123 | 2024-12-15 | "We chose X because..." |
| [E2] commit:def456 | 2024-12-18 | "Migrated to X for performance" |

## Related

- Skill: [skill-name](../../skills/skill-name/SKILL.md)
- Supersedes: ADR-NNN (if applicable)
```

## Storage Location

```
docs/adr/
├── 0001-use-hono-framework.md
├── 0002-adopt-dependency-injection.md
├── 0003-prefer-date-fns.md
└── index.md
```

### Index File

Auto-generated table of contents:

```markdown
# Architecture Decision Records

| ADR | Title | Date | Status |
|-----|-------|------|--------|
| 0001 | Use Hono Framework | 2024-12-15 | Accepted |
| 0002 | Adopt Dependency Injection | 2024-12-18 | Accepted |
| 0003 | Prefer date-fns over dayjs | 2024-12-20 | Accepted |
```

## Generation Workflow

### Step 1: Detect High-Confidence Pattern

During pattern extraction, check thresholds:

```python
def should_propose_adr(pattern):
    return (
        pattern.confidence >= 0.90 or
        len(pattern.evidence) >= 5 or
        pattern.frequency >= 5
    )
```

### Step 2: Extract ADR Content

Map pattern fields to ADR sections:

| Pattern Field | ADR Section |
|---------------|-------------|
| description | Context |
| core_pattern | Decision |
| evidence[].excerpt | Evidence table |
| constraints | Consequences (negative) |
| benefits | Consequences (positive) |

### Step 3: Present Proposal

```markdown
## ADR Proposal

Pattern "hono-framework" meets ADR threshold:
- Confidence: 92%
- Evidence: 6 sources
- Frequency: 8 occurrences

**Proposed ADR**:
- Title: Use Hono Framework for API Development
- Location: docs/adr/0004-use-hono-framework.md

Generate ADR? (yes/no/customize)
```

### Step 4: Generate or Customize

On approval:
1. Determine next ADR number
2. Generate ADR file
3. Update index.md
4. Link ADR in skill's Evidence Index

## Linking ADRs to Skills

### In Skill Files

Reference ADRs in Evidence Index:

```markdown
## Evidence Index

| ID | Source | Date | Excerpt |
|----|--------|------|---------|
| [E1] | ADR-0004 | 2024-12-20 | Architecture decision |
```

### In ADR Files

Link back to generating skill:

```markdown
## Related

- Skill: [api-endpoint-creation](../../.claude/skills/api-endpoint-creation/SKILL.md)
```

## Supersession Handling

When conflict resolution results in `supersede`:

1. Create new ADR with supersession note
2. Update old ADR status to "Superseded by ADR-NNN"
3. Link both in Related section

```markdown
# ADR-0003: Prefer date-fns over dayjs

**Status**: Accepted
**Supersedes**: ADR-0001 (dayjs adoption)

## Context

Team migrated from dayjs to date-fns for better tree-shaking...
```

## Lint Rules

| Rule | Description | Severity |
|------|-------------|----------|
| ADR-001 | High-confidence patterns should have ADR | Info |

**Example**:

```
ADR-001:1 - Pattern "hono-framework" (confidence: 92%) has no ADR
           Consider generating: docs/adr/0004-use-hono-framework.md
```

## Configuration

### Disable ADR Generation

In `.claude/settings.local.json`:

```json
{
  "skill-from-history": {
    "adr-generation": false
  }
}
```

### Custom Thresholds

```json
{
  "skill-from-history": {
    "adr-confidence-threshold": 0.85,
    "adr-evidence-threshold": 3
  }
}
```

### Custom Location

```json
{
  "skill-from-history": {
    "adr-directory": "architecture/decisions"
  }
}
```
