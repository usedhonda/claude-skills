---
name: skill-from-history
description: Analyzes Claude Code conversation history, git commits, and codebase to identify recurring patterns and generate project-specific skills. Invoke with "/skill-from-history" or when user asks to "create skills from history", "analyze patterns", "generate skills from logs", "suggest skills for this project".
---

# Skill from History Generator

A meta-skill that discovers recurring patterns from your project's history and generates reusable Claude Code skills. It analyzes multiple sources to identify both coding patterns and workflow patterns worth capturing.

## When to Use

Invoke this skill when:
- Starting work on a mature project to capture existing patterns
- After completing significant features to document patterns used
- Periodically to discover new patterns from recent work
- When team members want to share project-specific knowledge

**Trigger phrases**: "create skills from history", "analyze patterns", "generate skills", "suggest skills", "skill from history"

## Analysis Sources

This skill examines three complementary sources in priority order:

### 1. Claude Code Conversation History

**Location**: `~/.claude/projects/[encoded-project-path]/`

Files analyzed:
- `agent-[agentId].jsonl` - Agent-specific conversations
- `[sessionId].jsonl` - Session-specific conversations

**Extracted information**:
- User prompts and intent patterns
- Tool calls and their sequences
- Code changes and file modifications
- Repeated problem-solving approaches
- **Rejection patterns** (user corrections, "don't do X", "not like that")
- **Failed attempts** (assistant proposals rejected by user)

### 2. Git Commit History

**Commands used**:
```bash
git log --oneline -100
git log --format="%s" --since="30 days ago"
git diff HEAD~20..HEAD --stat
```

**Extracted information**:
- Commit message patterns and conventions
- File change frequencies
- Feature implementation patterns
- Bug fix approaches

### 3. Current Codebase

**Analyzed elements**:
- Directory structure and organization
- Code patterns in frequently modified files
- Configuration patterns
- Test patterns

## Execution Process

### Step 1: Identify Project Path

Determine the encoded project path for Claude Code history:
```
Current directory: /Users/example/projects/my-app
Encoded path: -Users-example-projects-my-app
History location: ~/.claude/projects/-Users-example-projects-my-app/
```

### Step 2: Gather Sources

Read and parse available data sources:

1. **Conversation history**: Parse JSONL files for tool calls, code blocks, and patterns
2. **Git history**: Extract commit messages and change patterns
3. **Codebase**: Scan for recurring structures and patterns

### Step 3: Check Existing Skills

Before proposing new skills, scan `.claude/skills/` for existing skills:

```bash
ls -la .claude/skills/
```

For each existing skill, read SKILL.md and extract:
- Skill name
- Description keywords
- Core patterns covered

### Step 4: Extract Pattern Candidates

Identify potential skills based on:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Frequency | 30% | Pattern appears in 3+ sessions/commits |
| Reusability | 25% | Contains extractable code templates |
| Complexity | 20% | Multi-step, non-trivial implementation |
| Documentation | 15% | Well-explained in history |
| Specificity | 10% | Project-specific value |

### Step 4.5: Extract Anti-patterns (Negative Learning)

Identify patterns the user has explicitly rejected or corrected. This unique feature learns "what NOT to do" from your correction history.

**Detection Signals**:

| Signal Type | Examples | Weight |
|-------------|----------|--------|
| Direct rejection | "No", "Don't", "Never use", "That's wrong" | High |
| Correction | "Actually...", "Instead of X, use Y", "Not like that" | High |
| Deprecation | "That's outdated", "We don't use X anymore" | Medium |
| Convention violation | "We always use X for this", "Follow our pattern" | Medium |
| Re-request | Same question asked again after assistant response | Low |

**Anti-pattern Structure**:

| Field | Description |
|-------|-------------|
| trigger | What caused the wrong response |
| wrong_approach | What was rejected |
| correct_approach | What should be done instead |
| reason | Why (if stated) |
| severity | critical / warning / info |

**Severity Classification**:

| Confidence | Frequency | Severity |
|------------|-----------|----------|
| > 80% | 3+ | Critical (must avoid) |
| > 60% | 2+ | Warning (discouraged) |
| > 40% | 1+ | Info (consider avoiding) |

For detailed detection algorithms, see [references/anti-pattern-detection.md](references/anti-pattern-detection.md).

### Step 5: Classify Patterns

Categorize each pattern into one of four categories:

**Development & Technical**:
- Error handling approaches
- API design conventions
- State management patterns
- Data validation methods
- Testing strategies

**Workflow & Process**:
- Deployment procedures
- Code review processes
- Testing workflows
- Build and release processes

**Creative & Design**:
- UI component patterns
- Styling conventions
- Animation patterns
- Layout templates

**Document & Data**:
- Documentation templates
- Data transformation patterns
- Report generation
- Configuration management

### Step 6: Conflict Detection

For each candidate, perform comprehensive conflict analysis:

**Similarity Check**:
- **Name similarity**: Levenshtein distance on skill names
- **Keyword overlap**: Common terms in descriptions
- **Pattern overlap**: Similar code structures

**Conflict Types**:

| Type | Severity | Description |
|------|----------|-------------|
| Duplicate | High | 80%+ similarity with existing skill |
| Overlap | Medium | 50-80% similarity, partial coverage |
| Missing Dependency | Medium | Pattern requires another skill |
| Outdated Reference | Low | References deprecated code/APIs |

**Resolution Options**:
- **Merge**: Combine with existing skill
- **Extend**: Add as variation to existing skill
- **Replace**: Supersede outdated skill
- **Skip**: Do not generate

### Step 7: Present Proposals

Display candidates in three categories:

```
## New Skill Candidates

1. [error-handling] (Confidence: 85%)
   Category: Coding Pattern
   Found in: 5 sessions, 12 commits
   Key pattern: Try-catch with custom error types
   Evidence: [E1][E2][E3]

2. [api-response-format] (Confidence: 78%)
   Category: Coding Pattern
   Found in: 8 sessions, 20 commits
   Key pattern: Consistent JSON response structure
   Evidence: [E4][E5]

## Additions to Existing Skills

3. [existing-skill] + 3 new patterns detected
   New patterns: validation-helper, cache-strategy, retry-logic
   Action: Merge into existing / Create separate

## Similar to Existing Skills

4. [new-deploy-process] ~ [deployment-workflow] (70% similar)
   Options: Merge / Create separate / Skip

## Detected Constraints (Negative Learning)

5. **[constraint-var-keyword]** (Confidence: 75%)
   - Severity: Warning
   - Rejected: `var` keyword
   - Use instead: `const` or `let`
   - Found in: 2 sessions
   - Evidence: [E6][E7]
```

### Step 7.5: Review Gate for Critical Constraints

When constraints meet Critical thresholds (confidence > 80%, frequency >= 3), they are marked as "Pending Critical" and require user approval before enforcement:

```
## Pending Critical Constraints (Requires Approval)

The following anti-patterns meet Critical threshold but require your confirmation:

### 1. [constraint-express-router]

- **Detected Pattern**: Don't use Express.js router
- **Use Instead**: Hono framework
- **Confidence**: 90%
- **Found in**: 3 sessions

**Evidence**:
- [E8] Session abc123: "No, we use Hono for all new endpoints"
- [E9] Session def456: "Don't use Express, switch to Hono"
- [E10] Session ghi789: "Express is deprecated in our codebase"

**Action Required**:
- `approve` - Confirm as Critical (blocks generation if violated)
- `warning` - Demote to Warning (shows warning but allows)
- `skip` - Remove this constraint

Your choice: [approve/warning/skip]
```

For detailed Review Gate workflow, see [references/anti-pattern-detection.md](references/anti-pattern-detection.md#review-gate).

### Step 8: Generate and Validate Skills

Based on user selection:

**For new skills**: Create complete skill directory:
```
.claude/skills/[skill-name]/
├── SKILL.md
└── references/
    └── patterns.md
```

**For existing skill updates**:
- Append patterns to `references/`
- Update relevant sections in SKILL.md
- Increment version if present

**Validation**: After generation, verify:

| Check | Requirement | Auto-fix |
|-------|-------------|----------|
| name | max 64 chars, lowercase, hyphens only | Truncate/convert |
| description | max 1024 chars, includes trigger phrases | Warn if missing |
| YAML frontmatter | Valid syntax | Error |
| Required sections | When to Use, Core Pattern | Warn |
| Word count | 1,500-2,000 words recommended | Info |

For detailed validation rules, see [references/validation.md](references/validation.md).

## Skill Generation Template

Generated skills follow this structure with evidence linking:

```yaml
---
name: [skill-name]
description: [Extracted description with trigger phrases from patterns]
---

# [Skill Title]

[Brief introduction based on extracted patterns] [E1][E2]

## When to Use

- [Trigger condition from history] [E1]
- [Another trigger from patterns] [E3]

## Core Pattern

[Main code pattern extracted from history] [E1][E4]

### Template

```[language]
[Reusable template from extracted patterns]
```

**Evidence**: Derived from [description] [E1], refined in [E4]

## Implementation Steps

1. [Step derived from history] [E2]
2. [Step derived from history] [E5]

## Variations

### [Variation Name]
[Alternative approach found in history] [E6]

## Best Practices

- [Practice observed in history] [E3]
- [Practice observed in history] [E7]

## Constraints (Auto-generated from negative learning)

The following patterns have been explicitly rejected in this project:

### Critical (Must avoid)

| Pattern | Instead Use | Reason | Evidence |
|---------|-------------|--------|----------|
| [Rejected pattern] | [Alternative] | [Why] | [E8][E9] |

### Warnings

| Pattern | Preferred | Reason | Evidence |
|---------|-----------|--------|----------|
| [Discouraged pattern] | [Better approach] | [Why] | [E10] |

---

## Evidence Index

Evidence supporting patterns and constraints in this skill.

### Sessions

| ID | Session | Date | Description |
|----|---------|------|-------------|
| E1 | `abc123` | 2024-12-15 | Initial pattern implementation |
| E3 | `def456` | 2024-12-18 | Trigger condition discussion |
| E8 | `ghi789` | 2024-12-20 | "Don't use X, we use Y instead" |

### Commits

| ID | Hash | Date | Message |
|----|------|------|---------|
| E2 | `a1b2c3d` | 2024-12-16 | feat: add pattern implementation |
| E4 | `e4f5g6h` | 2024-12-17 | refactor: optimize pattern |
| E5 | `i7j8k9l` | 2024-12-19 | docs: add step documentation |

### Code Patterns

| ID | File | Lines | Pattern |
|----|------|-------|---------|
| E6 | `src/example.ts` | 15-28 | Alternative approach template |
| E7 | `src/utils.ts` | 42-55 | Best practice implementation |
```

For evidence format specification, see [references/evidence-format.md](references/evidence-format.md).

## Example Session

```
User: /skill-from-history

Claude: Analyzing project history...

Found: 45 conversation sessions, 230 commits, 156 source files

## Analysis Results

### New Skill Candidates

1. **firebase-auth-flow** (Confidence: 92%)
   - Category: Development & Technical
   - Found in: 12 sessions, 25 commits
   - Pattern: Firebase authentication with custom claims
   - Includes: Login, logout, token refresh, role checking
   - Key Evidence:
     - [E1] Session abc123: "Use Firebase custom claims for roles"
     - [E2] Commit a1b2c3d: feat(auth): add custom claims handling

2. **api-error-handling** (Confidence: 85%)
   - Category: Development & Technical
   - Found in: 8 sessions, 18 commits
   - Pattern: Consistent error response format
   - Key Evidence:
     - [E3] Session def456: Error handling discussion
     - [E4] Commit e4f5g6h: fix(api): standardize error format

### Detected Constraints (Negative Learning)

3. **[constraint-express-router]** (Confidence: 90%)
   - Severity: Critical
   - Rejected: Express.js router
   - Use instead: Hono framework
   - Evidence:
     - [E5] Session ghi789: "No, we use Hono for all new endpoints"
     - [E6] Session jkl012: "Don't use Express, switch to Hono"
     - [E7] Session mno345: "Express is deprecated in our project"

### Existing Skill Updates

4. **[existing: deployment-workflow]** + 2 new patterns
   - New: Canary deployment steps, rollback procedure
   - Evidence: [E8] 3 sessions discussing deployment
   - Action needed: Merge or create separate

Which skills would you like to generate? (e.g., "1, 2" or "all")

User: 1

Claude: Generating firebase-auth-flow skill...

Created:
- .claude/skills/firebase-auth-flow/SKILL.md (1,847 words)
- .claude/skills/firebase-auth-flow/references/patterns.md

The skill includes:
- 8 evidence references (E1-E8)
- Evidence Index with sessions, commits, and code patterns
- 1 Critical constraint from negative learning

You can now use this skill by mentioning "Firebase auth" or
"authentication flow" in your prompts.
```

## Configuration

### Minimum Thresholds

Patterns are only proposed if they meet these minimums:
- Frequency: 3+ occurrences
- Confidence: 60% overall score

### Analysis Scope

Default analysis covers:
- Last 100 git commits
- All conversation history for project
- Active source files (excludes node_modules, build outputs)

## Troubleshooting

### No patterns found

If no patterns are detected:
1. Check if project has sufficient history
2. Verify Claude Code history exists at `~/.claude/projects/`
3. Ensure git repository has commits

### Too many similar suggestions

If suggestions overlap too much:
1. Increase similarity threshold
2. Focus on specific categories (coding vs workflow)
3. Run on specific time ranges

### Large history files

For projects with extensive history:
1. Limit analysis to recent sessions
2. Focus on specific file types
3. Use sampling for pattern detection

## Integration Notes

This skill generates skills that follow Claude Code's official specifications:
- SKILL.md: 1,500-2,000 words
- YAML frontmatter with name (max 64 chars) and description (max 1024 chars)
- Progressive disclosure via references/ directory

Generated skills are placed in the project's `.claude/skills/` directory and are immediately available for use.

For detailed pattern detection algorithms, see [references/pattern-detection.md](references/pattern-detection.md).
