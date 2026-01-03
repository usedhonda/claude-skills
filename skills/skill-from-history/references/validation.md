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
| Table format | Pattern, Instead Use/Preferred, Reason, Evidence columns |
| Severity order | Critical > Warnings > Info |
| Evidence linking | Critical constraints require 2+ evidence sources |

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

| Pattern | Instead Use | Reason | Evidence |
|---------|-------------|--------|----------|
| Express.js router | Hono framework | Project standard | [E5][E6][E7] |
| `var` keyword | `const`/`let` | ES6+ requirement | [E8][E9] |

### Warnings

| Pattern | Preferred | Reason | Evidence |
|---------|-----------|--------|----------|
| Inline styles | CSS modules | Maintainability | [E10] |
```

### Validation Output for Constraints

```
Constraints Section:
✓ Structure: valid (2 subsections)
✓ Critical: 2 patterns defined
✓ Warnings: 1 pattern defined
ℹ Info: not present (optional)
```

## File Size Limits

| File | Max Lines | Recommendation |
|------|-----------|----------------|
| SKILL.md | 500 | Keep under 400 for headroom |
| Reference files | 300 | Split if larger |

**Progressive Disclosure**: Move detailed content to `references/` directory. Claude loads these only when needed, reducing token usage.

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

## XREF: Cross-Reference Validation

Rules for validating agent-skill bidirectional references.

### XREF-001: agents: Reference Exists

**Severity**: Error

Skills referencing agents must have valid targets.

```yaml
# SKILL.md
agents:
  - name: security-auditor  # Must exist at .claude/agents/security-auditor/AGENT.md
```

**Validation**:
```
✓ agents[0].name 'security-auditor' → .claude/agents/security-auditor/AGENT.md exists
✗ agents[1].name 'nonexistent' → NOT FOUND
```

### XREF-002: Circular Reference Check

**Severity**: Error

Prevent Agent → Skill → Agent loops.

```
✗ Error: Circular reference detected
  security-auditor → secure-coding-patterns → security-auditor
```

### XREF-003: Declaration-Usage Match

**Severity**: Warning

Agents declared in frontmatter should be referenced in content.

```yaml
# Frontmatter
agents:
  - name: code-reviewer
  - name: security-auditor  # Warning: not referenced in content
```

```markdown
## Implementation Steps

### Step 2: Review
Use the `code-reviewer` agent for this phase.
# Note: security-auditor not mentioned → Warning
```

### XREF-004: skills: Reference in AGENT.md

**Severity**: Warning

Agents referencing skills should have valid targets.

```yaml
# AGENT.md
skills:
  - secure-coding-patterns  # Must exist at .claude/skills/secure-coding-patterns/SKILL.md
```

### XREF Validation Output

```
XREF Validation Results:

✓ XREF-001: All agent references valid (2/2)
✓ XREF-002: No circular references
⚠ XREF-003: 'security-auditor' declared but not used in content
✓ XREF-004: All skill references valid (1/1)

Status: PASS (1 warning)
```

---

## Extended Validation (Skill Lint)

For comprehensive validation including link checking, evidence validation, and structure analysis, see [skill-lint.md](skill-lint.md).

Skill Lint provides additional rules:
- **LINK**: Verify all links and evidence references resolve
- **EVID**: Validate Evidence Index completeness and format
- **CONS**: Ensure constraints have proper evidence backing
- **STRU**: Check markdown structure and hierarchy
- **CONT**: Detect placeholder text and TODO markers
- **XREF**: Validate agent-skill cross-references (see above)
