# Skill Lint Rules

Automated validation rules for generated skills. Run after skill generation to ensure quality and consistency.

## Rule Categories

### Link Validation (LINK)

| Rule | Description | Severity | Auto-fix |
|------|-------------|----------|----------|
| LINK-001 | All relative links must resolve to existing files | Error | No |
| LINK-002 | Evidence references `[E#]` must exist in Evidence Index | Error | No |
| LINK-003 | Anchor links `#section` must point to valid headers | Warning | No |

**Examples**:

```
LINK-001:45 - Broken link: [patterns.md](references/patterns.md)
             File not found: references/patterns.md

LINK-002:78 - Evidence [E15] not found in Evidence Index
             Referenced at line 78, but Evidence Index only contains E1-E12

LINK-003:92 - Invalid anchor: [section](#non-existent)
             Header "non-existent" not found in document
```

### Evidence Validation (EVID)

| Rule | Description | Severity | Auto-fix |
|------|-------------|----------|----------|
| EVID-001 | Evidence Index section required if any `[E#]` references used | Error | No |
| EVID-002 | Each evidence ID in Index must be referenced at least once | Warning | No |
| EVID-003 | Evidence excerpts should be < 100 characters | Info | Truncate |
| EVID-004 | Session IDs must be valid format (alphanumeric) | Error | No |
| EVID-005 | Commit hashes must be valid (7+ hex characters) | Error | No |

**Examples**:

```
EVID-001:1 - Missing Evidence Index section
             Document contains [E1], [E2], [E3] but no "## Evidence Index"

EVID-002:156 - Evidence E7 defined but never referenced
               Defined in Evidence Index but not used in document body

EVID-003:162 - Evidence excerpt too long (145 chars)
               "This is a very long excerpt that exceeds the 100 character..."
               Auto-fix: Truncate to 100 chars with "..."

EVID-004:158 - Invalid session ID format: "abc-123-!!!"
               Expected: alphanumeric characters only

EVID-005:160 - Invalid commit hash: "xyz123"
               Expected: 7+ hexadecimal characters (0-9, a-f)
```

### Constraint Validation (CONS)

| Rule | Description | Severity | Auto-fix |
|------|-------------|----------|----------|
| CONS-001 | Critical constraints require 2+ evidence sources | Error | No |
| CONS-002 | Each constraint must have non-empty "Instead Use" column | Warning | No |
| CONS-003 | Pending Critical constraints must have approval status | Error | No |
| CONS-004 | Constraint tables must have Evidence column | Warning | Add column |

**Examples**:

```
CONS-001:89 - Critical constraint "express-router" has only 1 evidence source
              Critical severity requires 2+ sources for validation

CONS-002:91 - Missing "Instead Use" for constraint "inline-styles"
              | inline-styles | | Maintainability |
                              ^-- Empty cell

CONS-003:95 - Pending Critical constraint missing approval status
              constraint-express-router: approval_status not set

CONS-004:88 - Constraint table missing Evidence column
              Expected columns: Pattern, Instead Use, Reason, Evidence
              Auto-fix: Add empty Evidence column
```

### Structure Validation (STRU)

| Rule | Description | Severity | Auto-fix |
|------|-------------|----------|----------|
| STRU-001 | Headers must follow hierarchy (no skipping levels) | Warning | Yes |
| STRU-002 | Code blocks must have language specifier | Info | No |
| STRU-003 | Tables must have header row | Warning | No |
| STRU-004 | No orphaned sections (content required after headers) | Warning | No |
| STRU-005 | Required sections must be present | Error | No |

**Required sections**:
- `# [Title]`
- `## When to Use`
- `## Core Pattern` or `## Implementation Steps`

**Examples**:

```
STRU-001:45 - Skipped heading level: ## -> ####
              Found #### after ##, expected ### first
              Auto-fix: Change #### to ###

STRU-002:78 - Code block without language specifier
              ```
              const x = 1
              ```
              Suggested: ```typescript or ```javascript

STRU-003:92 - Table missing header row
              | value1 | value2 |
              Expected header row with column names

STRU-004:105 - Orphaned section: ## Variations
               Header with no content before next header

STRU-005:1 - Missing required section: ## When to Use
             Document must contain "When to Use" section
```

### Content Validation (CONT)

| Rule | Description | Severity | Auto-fix |
|------|-------------|----------|----------|
| CONT-001 | No TODO/FIXME markers in final output | Warning | No |
| CONT-002 | No placeholder text (e.g., "[TBD]", "XXX", "...") | Warning | No |
| CONT-003 | Trigger phrases in description match "When to Use" section | Info | No |
| CONT-004 | Word count within recommended range | Info | No |

**Word count guidance**:
- Minimum: 500 words (Warning)
- Recommended: 1,500-2,000 words (OK)
- Maximum: 5,000 words (Warning)

**Examples**:

```
CONT-001:156 - TODO marker found: "TODO: add more examples"
               Remove or complete before finalizing

CONT-002:78 - Placeholder text: "[TBD]"
              Replace with actual content

CONT-003:3 - Trigger phrase mismatch
             Description mentions "create endpoint"
             "When to Use" section doesn't mention "create endpoint"

CONT-004:1 - Word count below minimum: 423 words
             Recommended: 1,500-2,000 words
```

## Execution

### Command Format

```
skill-lint <path-to-SKILL.md>
```

### Output Format

```
Linting api-endpoint/SKILL.md...

Errors (2):
  LINK-001:23 - Broken link: [patterns.md](references/patterns.md)
  EVID-002:156 - Evidence [E3] not found in Evidence Index

Warnings (3):
  STRU-001:45 - Skipped heading level: ## -> ####
  CONS-002:89 - Missing "Instead Use" for constraint "inline-styles"
  EVID-002:167 - Evidence [E7] defined but never referenced

Info (1):
  STRU-002:78 - Code block without language specifier

Summary: 2 errors, 3 warnings, 1 info
Status: FAIL (errors must be resolved)
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All checks passed |
| 1 | Warnings only (can proceed) |
| 2 | Errors found (must fix) |

## Auto-fix Support

Some rules support automatic fixing:

| Rule | Auto-fix Action |
|------|-----------------|
| EVID-003 | Truncate excerpts to 100 chars |
| CONS-004 | Add Evidence column to constraint tables |
| STRU-001 | Adjust header levels to follow hierarchy |

Run with `--fix` flag to apply auto-fixes:

```
skill-lint --fix <path-to-SKILL.md>
```

## Integration with Generation

Lint runs automatically after skill generation (Step 8):

1. **Pre-write validation**: Check structure before writing
2. **Post-write validation**: Verify links and evidence
3. **Report**: Display any issues found

| Check | When | Action on Fail |
|-------|------|----------------|
| Required sections | Pre-write | Block generation |
| Evidence links | Post-write | Error with suggestions |
| Constraint evidence | Post-write | Warning |
| Structure | Post-write | Auto-fix if possible |

## Configuration

### Disable Rules

In `.claude/settings.local.json`:

```json
{
  "skill-lint": {
    "disabled": ["STRU-002", "CONT-004"]
  }
}
```

### Custom Thresholds

```json
{
  "skill-lint": {
    "word-count-min": 300,
    "word-count-max": 3000,
    "excerpt-max-length": 150
  }
}
```
