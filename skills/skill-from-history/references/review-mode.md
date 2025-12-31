# skill-for-review: Constraint-Based Code Review

Review code changes against project-specific constraints extracted from history.

## Trigger

```
/skill-for-review [options]
```

**Aliases**: "review my changes", "check constraints", "validate diff"

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `--staged` | Review staged changes only | - |
| `--branch` | Review all changes on current branch vs main | - |
| `--pr <number>` | Review specific PR | - |
| `--files <glob>` | Filter to specific files | `*` |
| (none) | Review unstaged + staged changes | Yes |

## Execution Flow

### Step 1: Get Diff

```bash
# Default: all changes
git diff HEAD

# Staged only
git diff --staged

# Branch comparison
git diff main...HEAD

# PR (via gh)
gh pr diff <number>
```

### Step 2: Load Constraints

Scan all skills in `.claude/skills/` for Constraints sections:

```
.claude/skills/
├── api-endpoint-creation/
│   └── SKILL.md  → extract ## Constraints
├── error-handling/
│   └── SKILL.md  → extract ## Constraints
└── ...
```

Build constraint index:

```json
{
  "constraints": [
    {
      "id": "cons-001",
      "skill": "api-endpoint-creation",
      "severity": "critical",
      "pattern": "Express.js router",
      "instead_use": "Hono framework",
      "file_patterns": ["*.ts", "*.js"],
      "evidence": ["E5", "E6"]
    }
  ]
}
```

### Step 3: Match Constraints

For each changed file:

1. Parse diff hunks
2. Extract added/modified lines
3. Match against constraint patterns
4. Collect violations

**Matching logic**:

```python
def check_constraint(constraint, added_lines):
    matches = []
    for line_num, line in added_lines:
        if constraint.pattern in line:
            matches.append({
                "line": line_num,
                "content": line,
                "constraint": constraint
            })
    return matches
```

### Step 4: Generate Report

## Output Format

### Terminal Output

```
skill-for-review Results
========================

Analyzed: 5 files, 127 added lines
Constraints checked: 23 (8 Critical, 10 Warning, 5 Info)

## Violations (2)

### Critical

src/routes/users.ts:45
  Pattern: Express.js router
  Instead: Use Hono framework
  Skill: api-endpoint-creation [E5][E6]

  - import { Router } from 'express'
  + Consider: import { Hono } from 'hono'

### Warnings

src/utils/date.ts:12
  Pattern: dayjs import
  Instead: Use date-fns (superseded dayjs)
  Skill: date-handling [E15]
  ADR: docs/adr/0003-prefer-date-fns.md

## Applicable Constraints (5)

Constraints from skills that apply to changed files:

| Skill | Constraints | Severity |
|-------|-------------|----------|
| api-endpoint-creation | 3 | 2 Critical, 1 Warning |
| error-handling | 2 | 2 Warning |

## Summary

| Severity | Count | Status |
|----------|-------|--------|
| Critical | 1 | FAIL |
| Warning | 1 | WARN |
| Info | 0 | OK |

Result: FAIL (Critical violations must be resolved)
```

### JSON Output (--json)

```json
{
  "summary": {
    "files_analyzed": 5,
    "lines_added": 127,
    "constraints_checked": 23,
    "result": "fail"
  },
  "violations": [
    {
      "severity": "critical",
      "file": "src/routes/users.ts",
      "line": 45,
      "pattern": "Express.js router",
      "instead_use": "Hono framework",
      "skill": "api-endpoint-creation",
      "evidence": ["E5", "E6"]
    }
  ],
  "applicable_constraints": [
    {
      "skill": "api-endpoint-creation",
      "constraints": 3,
      "breakdown": {"critical": 2, "warning": 1}
    }
  ]
}
```

## Integration Points

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

result=$(claude /skill-for-review --staged --json)
if echo "$result" | jq -e '.summary.result == "fail"' > /dev/null; then
  echo "Critical constraint violations found. Commit blocked."
  echo "$result" | jq '.violations[] | select(.severity == "critical")'
  exit 1
fi
```

### CI Integration

```yaml
# .github/workflows/constraint-check.yml
name: Constraint Check
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check constraints
        run: |
          claude /skill-for-review --pr ${{ github.event.pull_request.number }}
```

### PR Comments

When run with `--pr`, can optionally post review comments:

```
/skill-for-review --pr 123 --comment
```

Posts inline comments at violation locations.

## Constraint Matching Details

### Pattern Types

| Type | Example | Matching |
|------|---------|----------|
| Literal | `Express.js` | String contains |
| Regex | `/import.*express/i` | Regex match |
| AST | `import { X } from 'express'` | AST pattern (future) |

### File Filtering

Constraints can specify file patterns:

```markdown
| Pattern | Instead Use | Files | Evidence |
|---------|-------------|-------|----------|
| express | hono | `src/routes/**` | [E5] |
```

Only checks constraints against matching files.

### Context Awareness

Some constraints are context-dependent:

```json
{
  "pattern": "any",
  "context": "TypeScript assertion",
  "scope": "production code only",
  "exclude": ["*.test.ts", "*.spec.ts"]
}
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | No violations |
| 1 | Warnings only |
| 2 | Critical violations |

## Configuration

### Ignore Patterns

In `.claude/settings.local.json`:

```json
{
  "skill-for-review": {
    "ignore": [
      "vendor/**",
      "generated/**"
    ]
  }
}
```

### Severity Override

```json
{
  "skill-for-review": {
    "severity-override": {
      "cons-001": "warning"  // Demote specific constraint
    }
  }
}
```

### Disable Skills

```json
{
  "skill-for-review": {
    "disabled-skills": ["legacy-patterns"]
  }
}
```
