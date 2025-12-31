# Evidence Format Specification

Evidence linking enables traceability from documented patterns/constraints back to their source.

## Evidence ID Format

### Syntax

```
[E<number>]
```

- `E` prefix (for "Evidence")
- Sequential number starting from 1
- Example: `[E1]`, `[E2]`, `[E15]`

### Generation Algorithm

```
evidence_id = "E" + sequential_counter

# Counter increments per skill generation
# Reset to 1 for each new skill
```

Evidence IDs are assigned in order of first appearance during pattern extraction:
1. Session evidence (chronological)
2. Commit evidence (chronological)
3. Code block evidence (file order)

## Evidence Types

### Session Evidence

Source: Claude Code conversation history (`~/.claude/projects/[encoded-path]/`)

```json
{
  "id": "E1",
  "type": "session",
  "session_id": "abc123def456",
  "timestamp": "2024-12-15T10:30:00Z",
  "excerpt": "Use Hono for all new API endpoints",
  "context": "User correction in API implementation discussion"
}
```

**Fields**:
| Field | Required | Description |
|-------|----------|-------------|
| id | Yes | Evidence ID (E1, E2, ...) |
| type | Yes | Always "session" |
| session_id | Yes | Claude Code session identifier |
| timestamp | Yes | ISO 8601 timestamp |
| excerpt | Yes | Relevant quote (max 100 chars) |
| context | No | Brief description of conversation context |

### Commit Evidence

Source: Git commit history (`git log`)

```json
{
  "id": "E2",
  "type": "commit",
  "hash": "a1b2c3d",
  "timestamp": "2024-12-17T14:20:00Z",
  "message": "refactor(api): migrate to Hono framework",
  "files_changed": ["src/api/router.ts", "src/api/handlers.ts"]
}
```

**Fields**:
| Field | Required | Description |
|-------|----------|-------------|
| id | Yes | Evidence ID |
| type | Yes | Always "commit" |
| hash | Yes | Short commit hash (7+ chars) |
| timestamp | Yes | Commit timestamp |
| message | Yes | Commit message (first line) |
| files_changed | No | List of changed files |

### Code Block Evidence

Source: Current codebase patterns

```json
{
  "id": "E3",
  "type": "code",
  "file": "src/api/router.ts",
  "lines": "15-28",
  "pattern": "Hono route handler with TypeScript types"
}
```

**Fields**:
| Field | Required | Description |
|-------|----------|-------------|
| id | Yes | Evidence ID |
| type | Yes | Always "code" |
| file | Yes | Relative file path |
| lines | Yes | Line range (start-end) |
| pattern | No | Brief pattern description |

## Rendering Rules

### Inline References

Use compact `[E#]` in body text:

```markdown
Use Hono's routing with TypeScript types [E1][E5]:
```

Multiple references can be grouped: `[E1][E2][E3]`

### In Tables

Evidence column in constraint tables:

```markdown
| Pattern | Instead Use | Reason | Evidence |
|---------|-------------|--------|----------|
| Express.js | Hono | Project standard | [E4][E5][E6] |
```

### Anchor Links (Optional)

For clickable references:

```markdown
[E1](#e1)
```

Links to corresponding entry in Evidence Index.

## Evidence Index Format

Located at the end of generated SKILL.md:

```markdown
---

## Evidence Index

Evidence supporting patterns and constraints in this skill.

### Sessions

| ID | Session | Date | Description |
|----|---------|------|-------------|
| E1 | `abc123` | 2024-12-15 | User stated Hono preference |
| E4 | `def456` | 2024-12-18 | "No, we use Hono for all new endpoints" |

### Commits

| ID | Hash | Date | Message |
|----|------|------|---------|
| E2 | `a1b2c3d` | 2024-12-17 | refactor(api): migrate to Hono framework |
| E3 | `e4f5g6h` | 2024-12-19 | feat(api): add user endpoints |

### Code Patterns

| ID | File | Lines | Pattern |
|----|------|-------|---------|
| E5 | `src/api/router.ts` | 15-28 | Hono route handler template |
```

## Constraints

### Maximum Evidence Count

- **Per skill**: 20 evidence entries maximum
- **Per pattern/constraint**: 5 evidence entries maximum

Rationale: Prevent bloat while maintaining traceability.

### Selection Priority

When evidence exceeds limits, prioritize:

1. **Recency**: Newer evidence preferred
2. **Explicitness**: Direct statements over implicit patterns
3. **Diversity**: Mix of sessions, commits, and code
4. **Relevance**: Higher confidence scores first

### Excerpt Length

- Session excerpts: Max 100 characters
- Commit messages: First line only
- Code patterns: Brief description, not full code

## Validation Rules

See `skill-lint.md` for validation:

- `EVID-001`: Evidence Index required if `[E#]` refs used
- `EVID-002`: All evidence IDs must be referenced
- `EVID-003`: Excerpts should be < 100 chars
- `EVID-004`: Session IDs must be valid format
- `EVID-005`: Commit hashes must be valid (7+ hex chars)
