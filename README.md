# skill-from-history

A Claude Code skill that analyzes your project's conversation history, git commits, and codebase to automatically generate reusable project-specific skills.

## Overview

This meta-skill discovers recurring patterns from your development work and generates Claude Code skills that capture your project's best practices, coding conventions, and workflow patterns.

### What it analyzes

1. **Claude Code Conversation History** - Past sessions, tool calls, and code changes
2. **Git Commit History** - Commit patterns, file changes, and feature implementations
3. **Current Codebase** - Directory structure, code patterns, and configurations

## Installation

Copy the `SKILL.md` file and `references/` directory to your Claude Code skills directory:

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills/skill-from-history

# Copy skill files
cp SKILL.md ~/.claude/skills/skill-from-history/
cp -r references ~/.claude/skills/skill-from-history/
```

Or for project-specific installation:

```bash
mkdir -p .claude/skills/skill-from-history
cp SKILL.md .claude/skills/skill-from-history/
cp -r references .claude/skills/skill-from-history/
```

## Usage

Invoke the skill in Claude Code with:

- `/skill-from-history`
- "create skills from history"
- "analyze patterns"
- "generate skills from logs"
- "suggest skills for this project"

### Example Session

```
User: /skill-from-history

Claude: Analyzing project history...

Found: 45 conversation sessions, 230 commits, 156 source files

## Analysis Results

### New Skill Candidates

1. **firebase-auth-flow** (Confidence: 92%)
   - Found in: 12 sessions, 25 commits
   - Pattern: Firebase authentication with custom claims

2. **api-error-handling** (Confidence: 85%)
   - Found in: 8 sessions, 18 commits
   - Pattern: Consistent error response format

Which skills would you like to generate?
```

## Configuration

### Pattern Detection Thresholds

- **Minimum frequency**: 3+ occurrences
- **Minimum confidence**: 60%

### Analysis Scope

- Last 100 git commits
- All Claude Code conversation history for the project
- Active source files (excludes node_modules, build outputs)

## How It Works

The skill identifies patterns based on:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Frequency | 30% | Pattern appears in 3+ sessions/commits |
| Reusability | 25% | Contains extractable code templates |
| Complexity | 20% | Multi-step, non-trivial implementation |
| Documentation | 15% | Well-explained in history |
| Specificity | 10% | Project-specific value |

## Generated Skill Structure

Skills are generated with:

```
.claude/skills/[skill-name]/
├── SKILL.md          # Main skill definition
└── references/
    └── patterns.md   # Detailed pattern documentation
```

## License

MIT License - see [LICENSE](LICENSE)

---

## 日本語

### 概要

`skill-from-history`は、Claude Codeの会話履歴、Gitコミット、コードベースを分析して、プロジェクト固有のスキルを自動生成するメタスキルです。

### インストール

```bash
# グローバルインストール
mkdir -p ~/.claude/skills/skill-from-history
cp SKILL.md ~/.claude/skills/skill-from-history/
cp -r references ~/.claude/skills/skill-from-history/

# プロジェクト固有インストール
mkdir -p .claude/skills/skill-from-history
cp SKILL.md .claude/skills/skill-from-history/
cp -r references .claude/skills/skill-from-history/
```

### 使い方

Claude Codeで以下のいずれかを入力:

- `/skill-from-history`
- 「履歴からスキルを作成」
- 「パターンを分析」

分析後、検出されたパターンから生成するスキルを選択できます。
