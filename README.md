# claude-skills

A collection of Claude Code skills for enhanced development workflows.

## Installation

### Via Plugin (Recommended)

```bash
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

## Available Skills

### skill-from-history

Analyzes your project's conversation history, git commits, and codebase to automatically generate reusable project-specific skills.

**Usage**:
- `/skill-from-history`
- "create skills from history"
- "analyze patterns"

[View documentation](skills/skill-from-history/SKILL.md)

## Structure

```
claude-skills/
├── .claude-plugin/
│   └── settings.json
└── skills/
    └── skill-from-history/
        ├── SKILL.md
        └── references/
```

## License

MIT License - see [LICENSE](LICENSE)

---

## 日本語

### インストール

```bash
# プラグイン経由（推奨）
/plugin install skill-from-history@usedhonda/claude-skills

# 手動インストール
mkdir -p ~/.claude/skills/skill-from-history
cp -r skills/skill-from-history/* ~/.claude/skills/skill-from-history/
```

### 収録スキル

- **skill-from-history**: 会話履歴・Git・コードベースからスキルを自動生成
