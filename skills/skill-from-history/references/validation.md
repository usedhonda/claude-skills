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
| `## Constraints` | Auto-generated from negative learning |

## Constraints Section Validation

When a Constraints section is present (auto-generated from negative learning):

### Structure Requirements

| Element | Requirement |
|---------|-------------|
| Subsections | At least one of: Critical, Warnings, Info |
| Table format | Pattern, Instead Use/Preferred, Reason columns |
| Severity order | Critical > Warnings > Info |

### Content Validation

| Check | Requirement |
|-------|-------------|
| Pattern field | Non-empty, describes what to avoid |
| Alternative field | Non-empty for Critical/Warnings |
| Reason field | Recommended but optional |

### Example Valid Constraints

```markdown
## Constraints (Auto-generated from negative learning)

### Critical (Must avoid)

| Pattern | Instead Use | Reason |
|---------|-------------|--------|
| Express.js router | Hono framework | Project standard |
| `var` keyword | `const`/`let` | ES6+ requirement |

### Warnings

| Pattern | Preferred | Reason |
|---------|-----------|--------|
| Inline styles | CSS modules | Maintainability |
```

### Validation Output for Constraints

```
Constraints Section:
✓ Structure: valid (2 subsections)
✓ Critical: 2 patterns defined
✓ Warnings: 1 pattern defined
ℹ Info: not present (optional)
```

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
