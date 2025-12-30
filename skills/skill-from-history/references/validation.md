# Skill Validation Rules

This document defines the validation rules for generated skills.

## YAML Frontmatter

### name (Required)

| Rule | Requirement |
|------|-------------|
| Max length | 64 characters |
| Format | lowercase, hyphens only |
| Pattern | `^[a-z][a-z0-9-]*$` |

**Auto-fix**:
- Convert to lowercase
- Replace spaces/underscores with hyphens
- Truncate if exceeds 64 chars

### description (Required)

| Rule | Requirement |
|------|-------------|
| Max length | 1024 characters |
| Must include | At least one trigger phrase |

**Validation**:
```
✓ "Generates API documentation. Invoke with /api-docs"
✗ "Generates API documentation" (missing trigger phrase)
```

## Required Sections

The following sections must exist in SKILL.md:

| Section | Purpose |
|---------|---------|
| `# [Title]` | Main heading matching skill name |
| `## When to Use` | Describes invocation conditions |
| `## Core Pattern` or equivalent | Main functionality |

## Recommended Sections

| Section | Purpose |
|---------|---------|
| `## Implementation Steps` | Step-by-step guide |
| `## Best Practices` | Usage guidelines |
| `## Troubleshooting` | Common issues |

## Word Count

| Level | Word Count | Status |
|-------|------------|--------|
| Minimum | 500 | Warning |
| Recommended | 1,500-2,000 | OK |
| Maximum | 5,000 | Warning |

## Validation Output

```
Validation Results for [skill-name]:

✓ name: valid (23 chars)
✓ description: valid (156 chars, 2 trigger phrases)
✓ YAML frontmatter: valid syntax
✓ Required sections: all present
ℹ Word count: 1,847 words (recommended range)

Status: PASS
```

## Error Handling

### Critical Errors (Generation fails)

- Invalid YAML syntax
- Missing `name` field
- Missing `description` field

### Warnings (Generation continues)

- Missing trigger phrases in description
- Missing recommended sections
- Word count outside recommended range

### Auto-corrections

- Name formatting (lowercase, hyphens)
- Whitespace normalization
- Duplicate section removal
