# claude-skills

A collection of Claude Code skills for enhanced development workflows.

## Installation

```bash
/plugin install skill-from-history@usedhonda/claude-skills
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
/plugin install skill-from-history@usedhonda/claude-skills
```

### 収録スキル

- **skill-from-history**: 会話履歴・Git・コードベースからスキルを自動生成
