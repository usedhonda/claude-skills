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

**Skill Conflict Types**:

| Type | Severity | Description |
|------|----------|-------------|
| Duplicate | High | 80%+ similarity with existing skill |
| Overlap | Medium | 50-80% similarity, partial coverage |
| Missing Dependency | Medium | Pattern requires another skill |
| Outdated Reference | Low | References deprecated code/APIs |

**Semantic Conflict Detection** (constraints):

| Conflict Type | Detection | Resolution |
|---------------|-----------|------------|
| Direct contradiction | Same subject, opposite polarity | User chooses |
| Temporal evolution | Newer supersedes older | `supersede` |
| Scope overlap | Same context, different actions | Define scopes |

Semantic conflicts require resolution before proceeding. For algorithm details, see [references/conflict-detection.md](references/conflict-detection.md).

**Resolution Options**:
- **Merge**: Combine with existing skill
- **Extend**: Add as variation to existing skill
- **Replace/Supersede**: Mark old as superseded, link to new
- **Scope**: Both valid in different contexts
- **Skip**: Do not generate

### Step 7: Present Proposals

Display candidates grouped by:
- **New Skill Candidates**: Patterns worthy of new skills
- **Additions to Existing Skills**: Extensions to current skills
- **Similar to Existing Skills**: Potential duplicates requiring decision
- **Detected Constraints**: Anti-patterns from negative learning

Each candidate shows confidence score, source count, and key evidence references.

### Step 7.5: Review Gate for Critical Constraints

Constraints meeting Critical thresholds (confidence > 80%, frequency >= 3) are marked "Pending Critical" and require user approval:
- `approve` - Confirm as Critical (blocks generation if violated)
- `warning` - Demote to Warning (shows warning but allows)
- `skip` - Remove constraint

For detailed workflow, see [references/anti-pattern-detection.md](references/anti-pattern-detection.md#review-gate).

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

## Skill Generation

Generated skills include:
- YAML frontmatter (name max 64 chars, description max 1024 chars)
- When to Use, Core Pattern, Constraints sections
- Evidence linking with `[E#]` references
- Evidence Index with sessions, commits, and code patterns

For complete template and examples, see [references/skill-template.md](references/skill-template.md).

## Glossary Extraction

During analysis, this skill automatically extracts project-specific terminology:

**Detection Signals**:
- Explicit definitions: "X means Y", "We call it X"
- Parenthetical expansions: "Use DI (Dependency Injection)"
- Corrections: "We call it Handler, not Controller"

**Output**: Terms are stored in `.claude/glossary.md` or skill-specific `glossary.md`.

**Integration**: Generated skills automatically include glossary notes for project-specific terms (e.g., "Handler (our term for Hono route controllers)").

For extraction patterns and format specification, see [references/glossary.md](references/glossary.md).

## ADR Generation

High-confidence patterns automatically trigger Architecture Decision Record (ADR) proposals.

**Trigger Thresholds** (any one):
- Confidence >= 90%
- Evidence count >= 5
- Frequency >= 5

**Output**: Lightweight ADRs in `docs/adr/` with:
- Context (why), Decision (what), Consequences
- Evidence table linking to source sessions/commits
- Bidirectional links to generating skill

**Supersession**: When constraints are superseded, ADRs are automatically updated with status changes and links.

For ADR format and workflow, see [references/adr-lite.md](references/adr-lite.md).

## Review Mode

Check code changes against project constraints before commit or PR.

**Trigger**: `/skill-for-review [--staged|--branch|--pr <number>]`

**Workflow**:
1. Get diff (staged, branch, or PR)
2. Load constraints from all skills in `.claude/skills/`
3. Match changed lines against constraint patterns
4. Report violations by severity

**Output**:
- Critical violations: Must resolve (blocks commit in hook mode)
- Warnings: Should resolve
- Applicable constraints: Shows relevant constraints for changed files

**Integration**: Can be used as pre-commit hook or CI step.

For detailed workflow and integration options, see [references/review-mode.md](references/review-mode.md).

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
