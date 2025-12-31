# Project Glossary

Specification for extracting and maintaining project-specific terminology.

## Term Extraction Signals

### Definition Patterns

| Signal | Pattern | Example |
|--------|---------|---------|
| Explicit definition | "X means Y" | "Handler means route controller in our project" |
| Parenthetical | "X (full name)" | "Use DI (Dependency Injection)" |
| Correction | "We call it X, not Y" | "We call it Handler, not Controller" |
| Context note | "X (our term for Y)" | "Handler (our term for Hono controllers)" |

### Japanese Patterns

| Signal | Pattern |
|--------|---------|
| Definition | "Xとは〜のこと" |
| Abbreviation | "X（〜の略）" |
| Clarification | "うちではXと呼ぶ" |

### Extraction Logic

```python
def extract_terms(message):
    terms = []

    # Definition pattern: "X means Y"
    if match := re.search(r'"?(\w+)"?\s+means?\s+(.+)', message):
        terms.append({
            "term": match.group(1),
            "definition": match.group(2),
            "type": "definition"
        })

    # Parenthetical: "X (full name)"
    if match := re.search(r'(\w+)\s+\(([^)]+)\)', message):
        terms.append({
            "term": match.group(1),
            "expansion": match.group(2),
            "type": "abbreviation"
        })

    # Correction: "We call it X, not Y"
    if match := re.search(r'call it\s+"?(\w+)"?,?\s+not\s+"?(\w+)"?', message):
        terms.append({
            "term": match.group(1),
            "not_term": match.group(2),
            "type": "correction"
        })

    return terms
```

## Glossary Format

### Storage Location

```
.claude/skills/[skill-name]/
└── glossary.md
```

Or project-wide:

```
.claude/
└── glossary.md
```

### File Format

```markdown
# Project Glossary

Auto-generated terminology dictionary.
Last updated: 2024-12-20

## Terms

### Handler

**Definition**: Route controller in Hono framework
**Context**: API layer
**Evidence**: [E5][E8]
**Note**: Different from generic "controller" - Hono-specific

### DI

**Full name**: Dependency Injection
**Usage**: "Use DI for service instantiation"
**Evidence**: [E3]
**Related skills**: service-layer-patterns

### FF

**Abbreviation for**: Feature Flag
**Usage**: "Check FF before enabling experimental features"
**Evidence**: [E12]

## Quick Reference

| Abbrev | Full Term | Context |
|--------|-----------|---------|
| DI | Dependency Injection | Architecture |
| FF | Feature Flag | Feature management |
| PR | Pull Request | Git workflow |
```

### Term Entry Structure

```json
{
  "term": "Handler",
  "type": "definition|abbreviation|correction",
  "definition": "Route controller in Hono framework",
  "expansion": null,
  "context": "API layer",
  "evidence": ["E5", "E8"],
  "related_skills": ["api-endpoint-creation"],
  "note": "Hono-specific",
  "extracted_at": "2024-12-20T10:30:00Z",
  "source_session": "abc123"
}
```

## Integration with Skill Generation

### Term Lookup

During skill generation:

1. Extract terms from pattern descriptions
2. Look up in glossary
3. Add glossary notes for project-specific terms

**Example transformation**:

```markdown
# Before (without glossary)
Use Handler for API routes

# After (with glossary lookup)
Use Handler (our term for Hono route controllers) for API routes
```

### Automatic Glossary Notes

When generating skills, add "See Glossary" notes:

```markdown
## Implementation

Use DI for service instantiation.

> **Note**: DI = Dependency Injection. See project glossary.
```

## Lint Rules

| Rule | Description | Severity |
|------|-------------|----------|
| GLOS-001 | Project-specific terms should be in glossary | Info |

**Example**:

```
GLOS-001:45 - Term "Handler" used but not in glossary
              Consider adding to glossary with /glossary-add
```

## Manual Management

### Add Term

```
/glossary-add "Handler" "Our term for Hono route controllers"
```

### Update Term

```
/glossary-update "Handler" --note "API layer only"
```

### Remove Term

```
/glossary-remove "OldTerm"
```

## Sync with Skills

When a term is added/updated:

1. Scan existing skills for the term
2. Suggest adding glossary notes
3. Update term's `related_skills` field

When a skill is generated:

1. Extract terms from content
2. Suggest new glossary entries
3. Add glossary references
