---
name: skill-from-history
description: Analyzes Claude Code conversation history, git commits, and codebase to identify recurring patterns and generate project-specific agents and skills. Invoke with "/skill-from-history:learn-gen" or when user asks to "create skills from history", "analyze patterns", "generate agents".
compression-anchors:
  - "Output: .claude/agents/[name]/AGENT.md + .claude/skills/[name]/SKILL.md"
  - "9-step process: Gather→Agents→Skills→Conflict→Propose→Generate"
  - "Governance: Maturity(Draft→Accepted→Canonical), Epochs, Staleness, Golden Tasks"
  - "Bidirectional: Agent→Skill (skills:) and Skill→Agent (agents:)"
  - "Global: ~/.claude/global-skills/ + Quarantine + Secret Redaction"
---

# Skill from History Generator

A meta-skill that discovers recurring patterns from your project's history and generates reusable Claude Code **agents** and **skills**. Agents define roles/perspectives; skills define workflows/procedures. Agents are generated first, then skills can reference them.

## TL;DR

**Input**: Claude Code history + Git commits + Codebase
**Output**: `.claude/agents/[name]/AGENT.md` + `.claude/skills/[name]/SKILL.md`
**Process**: 9 steps (Gather → Agents → Skills → Conflict → Propose → Generate)

**Quick Actions**:
- `/skill-from-history:learn-gen` - Generate agents + skills (recommended)
- `/skill-from-history:learn-gen --agents` - Agents only
- `/skill-from-history:learn-gen --skills` - Skills only

---

## When to Use

Invoke this skill when:
- Starting work on a mature project to capture existing patterns
- After completing significant features to document patterns used
- Periodically to discover new patterns from recent work
- When team members want to share project-specific knowledge

**Trigger phrases**: "create skills from history", "analyze patterns", "generate skills", "suggest skills", "skill from history"

---

## Analysis Sources

| Source | Location | Extracted Information |
|--------|----------|----------------------|
| **Conversation History** | `~/.claude/projects/[encoded-path]/` | User prompts, tool calls, rejections |
| **Git Commits** | `git log` | Commit patterns, file frequencies |
| **Codebase** | Source files | Directory structure, code patterns |

---

## Execution Process (9 Steps)

1. **Identify Project Path** - Find Claude Code history location
2. **Gather Sources** - Parse JSONL files, git history, codebase
3. **Check Existing** - Scan `.claude/skills/` and `.claude/agents/`
4. **Extract Agents** - Detect role/perspective patterns (60%+ confidence, 3+ occurrences)
5. **Extract Patterns** - Score by Frequency(30%), Reusability(25%), Complexity(20%), Documentation(15%), Specificity(10%)
6. **Extract Anti-patterns** - Detect rejected patterns ("No", "Don't", corrections)
7. **Conflict Detection** - Check duplicates (≥80% = update, 50-79% = extend, <50% = new)
8. **Present Proposals** - Display with confidence scores and recommendations
9. **Generate & Validate** - Create files with YAML frontmatter validation

詳細: [references/execution-details.md](references/execution-details.md)

---

## Generated Output

### Directory Structure

```
.claude/
├── agents/[name]/
│   └── AGENT.md
├── skills/[name]/
│   ├── SKILL.md
│   └── references/
│       └── patterns.md
└── commands/[skill-name].md  # Optional
```

### Skill Format

- YAML frontmatter: `name` (max 64 chars), `description` (max 1024 chars)
- Required sections: When to Use, Core Pattern, Constraints
- Evidence linking with `[E#]` references

詳細: [references/skill-template.md](references/skill-template.md), [references/agent-template.md](references/agent-template.md)

---

## Governance (v3.0)

### Constraint Maturity

| Level | 説明 | 強制力 |
|-------|------|--------|
| **Draft** | 提案段階 | Warning のみ |
| **Accepted** | 承認済み | Warning + CI Warning |
| **Canonical** | 標準規則 | Error + CI Block |
| **Deprecated** | 非推奨 | Info のみ |

昇格には: Eval Pass + Staleness Check + Rationale 記録が必要。

### Key Concepts

- **Epochs**: プロジェクトの時代区分（古いルールは適用されない）
- **Staleness**: 時間減衰 + 再確認 + 違反でスコア計算
- **Override**: `ALLOW_CONSTRAINT: CONS-XXX` + REASON で例外許可

詳細: [references/lifecycle.md](references/lifecycle.md)

### Golden Tasks (Eval Harness)

制約の有効性を検証するテストスイート:

```bash
/skill-from-history:learn-check            # 実行
/skill-from-history:learn-check --baseline # ベースライン比較
```

詳細: [references/golden-tasks.md](references/golden-tasks.md)

---

## Configuration

| Setting | Default |
|---------|---------|
| Minimum frequency | 3+ occurrences |
| Minimum confidence | 60% |
| Git history | Last 100 commits |
| Analysis scope | Active source files only |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No patterns found | Check history exists at `~/.claude/projects/` |
| Too many similar suggestions | Increase similarity threshold, focus categories |
| Large history files | Limit to recent sessions, use sampling |

---

## Commands

| コマンド | 説明 |
|---------|------|
| `/skill-from-history:learn-status` | 現在のステータス表示 |
| `/skill-from-history:learn-init` | Golden Tasks初期化 |
| `/skill-from-history:learn-gen` | Agent/Skill生成 |
| `/skill-from-history:learn-check` | 評価実行 |
| `/skill-from-history:learn-promote` | 成熟度昇格 |

---

## Key Reminders

- **Agent output**: `.claude/agents/[name]/AGENT.md` (generated first)
- **Skill output**: `.claude/skills/[name]/SKILL.md` (can reference agents)
- **Bidirectional**: Agent→Skill (`skills:`) and Skill→Agent (`agents:`)
- **Workflow**: Observe → Draft → Promote → Refine
- **Promotion rule**: Eval Pass + Staleness Check + Rationale required
- **Override**: `ALLOW_CONSTRAINT: CONS-XXX` with REASON
- **Global Skills**: `~/.claude/global-skills/` for cross-project sharing
- **Quarantine**: External skills start as Draft + `external: true`

---

## References

- [execution-details.md](references/execution-details.md) - Step-by-step process details
- [lifecycle.md](references/lifecycle.md) - Maturity, Epochs, Staleness, Override
- [golden-tasks.md](references/golden-tasks.md) - Evaluation harness
- [skill-template.md](references/skill-template.md) - SKILL.md format
- [agent-template.md](references/agent-template.md) - AGENT.md format
- [agent-detection.md](references/agent-detection.md) - Detection algorithms
- [anti-pattern-detection.md](references/anti-pattern-detection.md) - Negative learning
- [conflict-detection.md](references/conflict-detection.md) - Duplicate handling
- [validation.md](references/validation.md) - Validation rules
- [global-skills.md](references/global-skills.md) - Cross-Project Learning
