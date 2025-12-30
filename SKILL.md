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

### Step 5: Classify Patterns

Categorize each pattern as:

**Coding Patterns**:
- Error handling approaches
- API design conventions
- State management patterns
- Data validation methods
- Testing strategies

**Workflow Patterns**:
- Deployment procedures
- Code review processes
- Testing workflows
- Build and release processes

### Step 6: Compare with Existing Skills

For each candidate, check similarity with existing skills:

- **Name similarity**: Levenshtein distance on skill names
- **Keyword overlap**: Common terms in descriptions
- **Pattern overlap**: Similar code structures

### Step 7: Present Proposals

Display candidates in three categories:

```
## New Skill Candidates

1. [error-handling] (Confidence: 85%)
   Category: Coding Pattern
   Found in: 5 sessions, 12 commits
   Key pattern: Try-catch with custom error types

2. [api-response-format] (Confidence: 78%)
   Category: Coding Pattern
   Found in: 8 sessions, 20 commits
   Key pattern: Consistent JSON response structure

## Additions to Existing Skills

3. [existing-skill] + 3 new patterns detected
   New patterns: validation-helper, cache-strategy, retry-logic
   Action: Merge into existing / Create separate

## Similar to Existing Skills

4. [new-deploy-process] ~ [deployment-workflow] (70% similar)
   Options: Merge / Create separate / Skip
```

### Step 8: Generate or Update Skills

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

## Skill Generation Template

Generated skills follow this structure:

```yaml
---
name: [skill-name]
description: [Extracted description with trigger phrases from patterns]
---

# [Skill Title]

[Brief introduction based on extracted patterns]

## When to Use

- [Trigger condition from history]
- [Another trigger from patterns]

## Core Pattern

[Main code pattern extracted from history]

### Template

```[language]
[Reusable template from extracted patterns]
```

## Implementation Steps

1. [Step derived from history]
2. [Step derived from history]

## Variations

### [Variation Name]
[Alternative approach found in history]

## Best Practices

- [Practice observed in history]
- [Practice observed in history]

## Source References

- Session: [session-id] - [description]
- Commit: [commit-hash] - [message]
```

## Example Session

```
User: /skill-from-history

Claude: Analyzing project history...

Found: 45 conversation sessions, 230 commits, 156 source files

## Analysis Results

### New Skill Candidates

1. **firebase-auth-flow** (Confidence: 92%)
   - Found in: 12 sessions, 25 commits
   - Pattern: Firebase authentication with custom claims
   - Includes: Login, logout, token refresh, role checking

2. **api-error-handling** (Confidence: 85%)
   - Found in: 8 sessions, 18 commits
   - Pattern: Consistent error response format
   - Includes: Error types, logging, client responses

### Existing Skill Updates

3. **[existing: deployment-workflow]** + 2 new patterns
   - New: Canary deployment steps, rollback procedure
   - Action needed: Merge or create separate

Which skills would you like to generate? (e.g., "1, 2" or "all")

User: 1

Claude: Generating firebase-auth-flow skill...

Created:
- .claude/skills/firebase-auth-flow/SKILL.md (1,847 words)
- .claude/skills/firebase-auth-flow/references/patterns.md

The skill captures:
- Authentication flow with Firebase
- Custom claims management
- Token refresh handling
- Role-based access patterns

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
