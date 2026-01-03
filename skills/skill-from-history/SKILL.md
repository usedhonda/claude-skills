---
name: skill-from-history
description: Analyzes Claude Code conversation history, git commits, and codebase to identify recurring patterns and generate project-specific agents and skills. Invoke with "/gen-all", "/gen-agents", "/gen-skills", or when user asks to "create skills from history", "analyze patterns", "generate agents".
compression-anchors:
  - "Output: .claude/agents/[name]/AGENT.md + .claude/skills/[name]/SKILL.md"
  - "9-step process: Gather→Agents→Skills→Conflict→Propose→Generate"
  - "Governance: Maturity(Draft→Accepted→Canonical), Epochs, Staleness, Golden Tasks"
  - "Bidirectional: Agent→Skill (skills:) and Skill→Agent (agents:)"
---

# Skill from History Generator

A meta-skill that discovers recurring patterns from your project's history and generates reusable Claude Code **agents** and **skills**. Agents define roles/perspectives; skills define workflows/procedures. Agents are generated first, then skills can reference them.

## TL;DR

**Input**: Claude Code history + Git commits + Codebase
**Output**: `.claude/agents/[name]/AGENT.md` + `.claude/skills/[name]/SKILL.md`
**Process**: 9 steps (Gather → Agents → Skills → Conflict → Propose → Generate)

**Quick Actions**:
- `/gen-all` - Generate agents + skills (recommended)
- `/gen-agents` - Agents only
- `/gen-skills` - Skills only

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

Also scan `.claude/agents/` for existing agents.

### Step 3.5: Extract Agent Patterns

Detect recurring role/perspective patterns from conversation history:

**Detection Signals** (weighted):
| Signal | Weight | Example |
|--------|--------|---------|
| Role descriptions | 35% | "As a security expert..." |
| Perspective consistency | 30% | Same viewpoint across sessions |
| Tool usage patterns | 20% | Grep + Read combinations |
| Task tool invocations | 15% | Similar prompts repeated |

**Threshold**: 60% confidence, 3+ occurrences

**Output**: Agent candidates with role, perspective, prompt template.

See [references/agent-detection.md](references/agent-detection.md) for algorithms.

### Step 4: Extract Pattern Candidates

Score patterns by weighted criteria: Frequency(30%), Reusability(25%), Complexity(20%), Documentation(15%), Specificity(10%).

### Step 4.5: Extract Anti-patterns (Negative Learning)

Detect patterns the user rejected ("No", "Don't", corrections). Classify by severity: Critical (>80% confidence, 3+ occurrences), Warning, Info.

See [references/anti-pattern-detection.md](references/anti-pattern-detection.md) and [references/execution-details.md](references/execution-details.md).

### Step 5: Classify Patterns

Categorize into: Development & Technical, Workflow & Process, Creative & Design, Document & Data.

### Step 6: Conflict Detection & Update Proposal

Check for duplicates, overlaps, and semantic conflicts. **Also detect updates to existing agents/skills**.

**判定フロー**:
```
>= 80% 類似 → 新パターンあり? → 更新提案 / スキップ
50-79% 類似 → 拡張提案（別名で新規）
< 50% → 新規生成
```

**更新提案の表示**:
```
## Existing Updates

| Type | Name | Match | New Patterns | Action |
|------|------|-------|--------------|--------|
| Agent | security-auditor | 85% | +2 evidence | Update? |
| Skill | multi-ai-review | 88% | +1 step | Update? |
```

See [references/conflict-detection.md](references/conflict-detection.md) and [references/agent-detection.md](references/agent-detection.md).

### Step 7: Present Proposals

Display grouped candidates with confidence scores and command recommendations (✓ Recommend / △ Optional / - Skip).

### Step 7.5: Review Gate

Critical constraints (>80% confidence, 3+ frequency) require user approval before enforcement.

See [references/execution-details.md](references/execution-details.md) for detailed tables and criteria.

### Step 8: Generate and Validate Skills

Based on user selection:

**For new skills**: Create complete skill directory:
```
.claude/skills/[skill-name]/
├── SKILL.md
└── references/
    └── patterns.md
```

**For skills with Command recommendation**: Also create command file:
```
.claude/commands/[skill-name].md
```

Command file format:
```markdown
---
description: [Brief skill description]
---

# [Skill Title]

[When to Use summary]

Usage:
- `/[skill-name]` - [Primary use case]
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

High-confidence patterns trigger Architecture Decision Record (ADR) proposals via a 3-stage Human-in-the-Loop workflow.

**Trigger Thresholds** (any one):
- Confidence >= 90%
- Evidence count >= 5
- Frequency >= 5

**3-Stage Workflow**:
1. **Proposal**: Pattern detected, user prompted (no file created)
2. **Draft**: User approves → Draft ADR in `docs/adr/drafts/` (editable, not enforced)
3. **Accept**: User finalizes → Moved to `docs/adr/`, enforced in review

**Why Human-in-the-Loop**: Prevents accidental formalization of anti-patterns or exploratory conversations as project standards.

For ADR format and workflow details, see [references/adr-lite.md](references/adr-lite.md).

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

## Governance (v3.0)

制約とスキルのライフサイクル管理。

### Constraint Maturity

制約は成熟度レベルを持つ：

| Level | 説明 | 強制力 |
|-------|------|--------|
| **Draft** | 提案段階 | Warning のみ |
| **Accepted** | 承認済み | Warning + CI Warning |
| **Canonical** | 標準規則 | Error + CI Block |
| **Deprecated** | 非推奨 | Info のみ |

昇格には: Eval Pass + Staleness Check + Rationale 記録が必要。

### Epochs

プロジェクトの時代区分。古い Epoch のルールは現在では適用されない。

```yaml
epochs:
  - id: E1
    name: monolith-era
    status: deprecated
  - id: E2
    name: microservices-era
    status: active
current_epoch: E2
```

### Staleness & Drift

- **Staleness Score**: 時間減衰 + 再確認 + 違反回数で計算（0-1）
- **Drift Detection**: パターン消失、違反急増、頻繁なオーバーライドを検出
- **TTL**: 制約には有効期限があり、期限切れで自動 Deprecation

### Override Protocol

制約を例外的に無視する場合：

```
ALLOW_CONSTRAINT: CONS-001
REASON: Legacy compatibility required
```

Override は記録され、Staleness スコアに影響する。

### Golden Tasks (Eval Harness)

制約の有効性を検証するテストスイート：

```bash
$ /skill-eval           # Golden Tasks 実行
$ /skill-eval --baseline  # ベースライン比較
$ /skill-eval --regression CONS-NEW  # 回帰テスト
```

詳細は [references/lifecycle.md](references/lifecycle.md) と [references/golden-tasks.md](references/golden-tasks.md) を参照。

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

## Key Reminders

Essential points for context retention:

- **Agent output**: `.claude/agents/[name]/AGENT.md` (generated first)
- **Skill output**: `.claude/skills/[name]/SKILL.md` (can reference agents)
- **Bidirectional**: Agent→Skill (`skills:` field) and Skill→Agent (`agents:` field)
- **Commands**: `/gen-all` (both), `/gen-agents`, `/gen-skills`, `/skill-eval`
- **Governance**: Maturity (Draft→Accepted→Canonical→Deprecated), Epochs, Staleness
- **Promotion rule**: Eval Pass + Staleness Check + Rationale required
- **Override**: `ALLOW_CONSTRAINT: CONS-XXX` with REASON
- **Validation**: agent-lint + skill-lint + XREF rules + Golden Tasks
- **Evidence**: All patterns linked to source via `[E#]` references

**Reference docs**:
- [lifecycle.md](references/lifecycle.md) - Maturity, Epochs, Staleness, Override
- [golden-tasks.md](references/golden-tasks.md) - Evaluation harness
- [agent-template.md](references/agent-template.md) - AGENT.md format
- [agent-detection.md](references/agent-detection.md) - Detection algorithms
- [agent-lint.md](references/agent-lint.md) - Validation rules
