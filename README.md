# claude-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

A collection of Claude Code skills for enhanced development workflows.

## Features

### skill-from-history

Automatically discovers patterns from your development history and generates reusable skills.

- **Multi-source analysis**: Conversation history, Git commits, and codebase
- **4 pattern categories**: Development, Workflow, Creative, Document
- **Conflict detection**: Prevents duplicate skills
- **Output validation**: Ensures generated skills meet specifications

## Requirements

- [Claude Code](https://claude.ai/claude-code) (Pro, Max, Team, or Enterprise)
- Git repository (recommended for full analysis)

## Installation

### Via Plugin (Recommended)

```bash
# Add marketplace first
/plugin marketplace add usedhonda/claude-skills

# Then install
/plugin install skill-from-history@usedhonda/claude-skills
```

### Manual Installation

```bash
# Global
mkdir -p ~/.claude/skills/skill-from-history
cp -r skills/skill-from-history/* ~/.claude/skills/skill-from-history/

# Project-specific
mkdir -p .claude/skills/skill-from-history
cp -r skills/skill-from-history/* .claude/skills/skill-from-history/
```

## How It Works

```
1. Analyze    →  2. Detect     →  3. Propose    →  4. Generate
   Sources         Patterns         Candidates       Skills

- Conversations   - Frequency      - Confidence     - SKILL.md
- Git history     - Reusability    - Category       - references/
- Codebase        - Complexity     - Conflicts
```

## Usage

```bash
/skill-from-history
```

Or say: "create skills from history", "analyze patterns", "suggest skills"

[View full documentation](skills/skill-from-history/SKILL.md)

## Available Skills

| Skill | Description |
|-------|-------------|
| [skill-from-history](skills/skill-from-history/SKILL.md) | Generate skills from project history |

## Contributing

Issues and PRs are welcome!

## License

MIT License - see [LICENSE](LICENSE)

---

## 日本語

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

Claude Code用スキルコレクション

### 特徴

- **マルチソース分析**: 会話履歴、Git、コードベース
- **4カテゴリ分類**: Development, Workflow, Creative, Document
- **重複検出**: 既存スキルとの競合を防止
- **出力検証**: 仕様準拠を保証

### 必要条件

- Claude Code（Pro, Max, Team, Enterprise）
- Gitリポジトリ（推奨）

### インストール

```bash
# プラグイン経由（推奨）
/plugin marketplace add usedhonda/claude-skills
/plugin install skill-from-history@usedhonda/claude-skills

# 手動
mkdir -p ~/.claude/skills/skill-from-history
cp -r skills/skill-from-history/* ~/.claude/skills/skill-from-history/
```

### 使い方

```bash
/skill-from-history
```

または「履歴からスキル作成」「パターン分析」と入力
