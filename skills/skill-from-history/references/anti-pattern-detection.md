# Anti-pattern Detection Reference

Detailed algorithms for identifying rejected patterns and generating constraints from conversation history.

## Overview

Anti-pattern detection (Negative Learning) identifies patterns that users have explicitly rejected or corrected. Unlike positive pattern extraction that learns "what to do", this learns "what NOT to do" - creating guardrail skills that prevent repeated mistakes.

## Rejection Signal Detection

### Linguistic Patterns

| Category | Patterns | Confidence | Examples |
|----------|----------|------------|----------|
| Direct negation | "No", "Don't", "Never", "Stop" | 0.95 | "No, don't use var" |
| Correction | "Actually", "Instead", "Rather" | 0.85 | "Actually, use const here" |
| Preference | "We prefer", "We always", "Our convention" | 0.75 | "We always use Hono" |
| Deprecation | "Outdated", "Old way", "Don't use anymore" | 0.80 | "That's the old API" |
| Quality | "That's wrong", "Incorrect", "Bad practice" | 0.90 | "That's a bad practice" |

### Japanese Patterns

| Category | Patterns | Confidence |
|----------|----------|------------|
| Direct negation | "違う", "やめて", "使わないで", "ダメ" | 0.95 |
| Correction | "そうじゃなくて", "代わりに", "むしろ" | 0.85 |
| Preference | "うちでは", "いつも〜する", "慣例では" | 0.75 |
| Deprecation | "古い", "非推奨", "もう使わない" | 0.80 |

## Contextual Analysis

### Sequence Detection

A valid anti-pattern requires this conversation sequence:

```
1. Assistant proposes code/approach
   └─ Contains: code block, suggestion, implementation

2. User responds with rejection signal
   └─ Contains: negation keywords, correction phrases

3. (Optional) User provides correct alternative
   └─ Contains: "instead", "use X", code block

4. (Optional) Assistant acknowledges and changes
   └─ Confirms the rejection was accepted
```

### Multi-turn Pattern Detection

```
If same rejection appears across 2+ sessions:
  severity = "critical"
  confidence_boost = +0.2

If rejection appears in recent 7 days:
  recency_boost = +0.1
```

### Re-request Detection

When user asks the same question after receiving a response:

```
Pattern:
  User: "How do I create an API endpoint?"
  Assistant: [provides Express.js example]
  User: "How do I create an API endpoint with Hono?"

Detection:
  - Same intent, different specification
  - Implies first response was unsatisfactory
  - Confidence: 0.6 (lower than explicit rejection)
```

## Anti-pattern Categories

### 1. Forbidden Libraries/Frameworks

**Detection signals**:
- "Don't use X library"
- "We've migrated from X to Y"
- "X is not allowed in this project"

**Extraction**:
```
Look for: package names, import statements, framework references
in rejection context
```

**Example**:
```json
{
  "type": "forbidden",
  "wrong": "moment.js",
  "correct": "date-fns",
  "reason": "Bundle size and tree-shaking"
}
```

### 2. Deprecated Syntax/APIs

**Detection signals**:
- "That API is deprecated"
- "Use the new syntax"
- "That's the old way"

**Extraction**:
```
Look for: version numbers, API names, syntax patterns
Compare: against known deprecation lists
```

**Example**:
```json
{
  "type": "deprecated",
  "wrong": "componentWillMount",
  "correct": "useEffect",
  "reason": "React 18+ lifecycle"
}
```

### 3. Convention Violations

**Detection signals**:
- "We always use X naming"
- "Follow our pattern for Y"
- "That doesn't match our style"

**Extraction**:
```
Look for: naming patterns, file structure references, style mentions
```

**Example**:
```json
{
  "type": "convention",
  "wrong": "camelCase for files",
  "correct": "kebab-case for files",
  "reason": "Project naming convention"
}
```

### 4. Architecture Constraints

**Detection signals**:
- "That belongs in X layer"
- "Don't put logic there"
- "That's not where we put X"

**Extraction**:
```
Look for: directory paths, layer names (controller, service, model),
architectural terms
```

**Example**:
```json
{
  "type": "architecture",
  "wrong": "Business logic in controller",
  "correct": "Business logic in service layer",
  "reason": "Clean architecture"
}
```

### 5. Security/Quality Concerns

**Detection signals**:
- "That's a security risk"
- "Don't expose X"
- "That could cause Y problem"

**Extraction**:
```
Look for: security keywords (expose, leak, vulnerability),
quality terms (performance, memory, race condition)
```

**Example**:
```json
{
  "type": "security",
  "wrong": "Storing API key in code",
  "correct": "Use environment variables",
  "reason": "Security best practice"
}
```

## Scoring Algorithm

### Rejection Signal Score

| Signal Type | Base Score |
|-------------|------------|
| Explicit "don't"/"never" | 1.0 |
| Correction with alternative | 0.9 |
| Preference statement | 0.7 |
| Re-request after response | 0.6 |
| Negative sentiment only | 0.4 |

### Anti-pattern Confidence Calculation

```
confidence = (
  rejection_signal * 0.40 +
  frequency * 0.30 +
  recency * 0.20 +
  explicitness * 0.10
)
```

Where:
- `rejection_signal`: Score from table above
- `frequency`: `min(occurrences / 3, 1.0)`
- `recency`: `1.0` if within 7 days, `0.7` if within 30 days, `0.4` otherwise
- `explicitness`: `1.0` if alternative provided, `0.5` if reason given, `0.2` otherwise

### Severity Classification

| Confidence | Frequency | Severity | Action |
|------------|-----------|----------|--------|
| > 80% | 3+ | Pending Critical | Requires user approval (see Review Gate) |
| > 60% | 2+ | Warning | Warn user, suggest alternative |
| > 40% | 1+ | Info | Include in documentation |

**Note**: Critical severity is only assigned after user approval through the Review Gate process.

## Output Format

### Constraint Structure

```json
{
  "id": "constraint-[hash]",
  "type": "forbidden|deprecated|convention|architecture|security",
  "severity": "pending_critical|critical|warning|info",
  "approval_status": "pending|approved|rejected",
  "confidence": 85,
  "frequency": 3,
  "pattern": {
    "trigger": "When creating API endpoints",
    "wrong": "Using Express.js router",
    "correct": "Use Hono framework",
    "reason": "Project standard since 2024"
  },
  "sources": [
    {
      "evidence_id": "E5",
      "type": "session",
      "id": "abc123",
      "timestamp": "2024-12-15T10:30:00Z",
      "excerpt": "No, we use Hono for all new endpoints"
    },
    {
      "evidence_id": "E6",
      "type": "session",
      "id": "def456",
      "timestamp": "2024-12-20T14:15:00Z",
      "excerpt": "Don't use Express, switch to Hono"
    }
  ],
  "target_skill": "api-endpoint-creation"
}
```

### Generated Constraints Section

```markdown
## Constraints (Auto-generated from negative learning)

The following patterns have been explicitly rejected in this project:

### Critical (Must avoid)

| Pattern | Instead Use | Reason | Evidence |
|---------|-------------|--------|----------|
| Express.js router | Hono framework | Project standard | [E5][E6][E7] |
| `var` keyword | `const`/`let` | ES6+ requirement | [E8][E9] |

### Warnings

| Pattern | Preferred | Reason | Evidence |
|---------|-----------|--------|----------|
| Inline styles | CSS modules | Maintainability | [E10] |

### Info

- Consider using TypeScript strict mode [E11]
- Prefer named exports over default exports [E12]
```

## Integration with Skills

### New Skill Generation

When generating a new skill, constraints are automatically included:

```markdown
## [Skill Name]

... (normal content) ...

## Constraints

[Auto-generated constraints relevant to this skill's domain]
```

### Existing Skill Updates

When updating an existing skill with new constraints:

1. Read existing Constraints section (if any)
2. Merge new constraints, avoiding duplicates
3. Update severity if confidence increased
4. Add new sources to existing constraints

## Conflict Resolution

### Priority Rules

| Situation | Resolution | Rationale |
|-----------|------------|-----------|
| Same pattern: positive & negative | Negative wins | Safety first |
| Recent negative, old positive | Negative wins | Latest knowledge |
| Team-wide negative, personal positive | Team wins | Consistency |
| Conflicting negatives | Higher confidence wins | More evidence |

### Edge Cases

**Temporary constraints**:
```
If rejection mentions "for now", "temporarily", "until X":
  Mark constraint as temporary
  Add expiration context
```

**Conditional constraints**:
```
If rejection is context-specific:
  "Don't use X in production" (OK in dev)
  Add condition field to constraint
```

## Performance Considerations

### Efficient Scanning

1. **First pass**: Keyword matching for rejection signals
2. **Second pass**: Context analysis only for matches
3. **Caching**: Store extracted anti-patterns per session

### Large History Handling

- Process sessions in reverse chronological order
- Stop after finding 10 high-confidence anti-patterns
- Sample older sessions (every 5th) for frequency validation

## Review Gate

### Purpose

Critical severity constraints can block code generation and significantly impact workflow. To prevent false positives from becoming blockers, constraints that meet Critical thresholds are initially marked as "Pending Critical" and require human approval before becoming enforced Critical constraints.

### Workflow

```
┌─────────────────────────────────────────────────────────┐
│              Anti-pattern Detection                      │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
        ┌─────────────────────────────────────┐
        │  Auto-classify: Warning or Info     │
        │  (based on confidence/frequency)    │
        └────────────────────┬────────────────┘
                             │
          ┌──────────────────┴──────────────────┐
          │ Confidence > 80% && Frequency >= 3? │
          └──────────────────┬──────────────────┘
                             │ Yes
                             ▼
        ┌─────────────────────────────────────┐
        │   Mark as "Pending Critical"        │
        │   (approval_status: pending)        │
        └────────────────────┬────────────────┘
                             │
                             ▼
        ┌─────────────────────────────────────┐
        │   Present to user for approval      │
        │   with evidence excerpts            │
        └────────────────────┬────────────────┘
                             │
          ┌──────────────────┴──────────────────┐
          │        User response?               │
          └──────────────────┬──────────────────┘
               │ Approve           │ Reject/Demote
               ▼                   ▼
    ┌──────────────────┐     ┌──────────────────────┐
    │ Promote to       │     │ Set to Warning       │
    │ Critical         │     │ or remove constraint │
    │ (status:approved)│     │ (status: rejected)   │
    └──────────────────┘     └──────────────────────┘
```

### Presentation Format

When Pending Critical constraints are detected, present them for approval:

```markdown
## Pending Critical Constraints (Requires Approval)

The following anti-patterns meet Critical threshold but require your confirmation:

### 1. [constraint-express-router]

- **Detected Pattern**: Don't use Express.js router
- **Use Instead**: Hono framework
- **Confidence**: 90%
- **Found in**: 3 sessions

**Evidence**:
- [E5] Session abc123: "No, we use Hono for all new endpoints"
- [E6] Session def456: "Don't use Express, switch to Hono"
- [E7] Session ghi789: "Express is deprecated in our codebase"

**Action Required**:
- `approve` - Confirm as Critical (blocks generation if violated)
- `warning` - Demote to Warning (shows warning but allows)
- `skip` - Remove this constraint

Your choice: [approve/warning/skip]
```

### Approval Persistence

Approved constraints are stored with their approval metadata:

```json
{
  "id": "constraint-express-router",
  "severity": "critical",
  "approval_status": "approved",
  "approved_at": "2024-12-20T15:30:00Z",
  "approved_by": "user",
  "original_confidence": 90
}
```

### Automatic Re-review

Previously approved constraints may require re-review if:

- Confidence drops below 70%
- No new evidence in 90 days
- Conflicting positive patterns detected

In these cases, the constraint is demoted to "pending_review" status.
