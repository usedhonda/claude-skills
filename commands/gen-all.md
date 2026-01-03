---
description: エージェントとスキルを統合生成（エージェント優先）
argument-hint: [options]
---

# Agent & Skill Generator

Claude Code の会話履歴を分析し、**エージェント**と**スキル**の両方を生成。

エージェントを先に生成し、スキルがエージェントを参照できるようにする。

## 実行プロセス

```
1. ソース収集
   └── 会話履歴、Git、コードベース

2. エージェントパターン抽出
   └── 役割/ペルソナの検出

3. スキルパターン抽出
   └── ワークフロー/手順の検出

4. 競合検出
   └── エージェント間、スキル間、エージェント-スキル間

5. 提案表示
   ├── エージェント候補（先に表示）
   └── スキル候補（エージェント参照可能）

6. ユーザー選択

7. 生成
   ├── エージェント生成（先に）
   └── スキル生成（エージェント参照可能）

8. バリデーション
   └── agent-lint + skill-lint + XREF
```

## 提案表示例

```markdown
## Analysis Results

### Agent Candidates

| # | Agent | Confidence | Reason |
|---|-------|------------|--------|
| A1 | security-auditor | 85% | 6 sessions with security focus |
| A2 | api-designer | 72% | API design discussions |

### Skill Candidates

| # | Skill | Confidence | Uses Agent |
|---|-------|------------|------------|
| S1 | security-review-workflow | 88% | A1: security-auditor |
| S2 | api-design-check | 75% | A2: api-designer |

Select to generate:
- Agents: A1, A2
- Skills: S1, S2
```

## 出力

```
.claude/
├── agents/
│   ├── security-auditor/
│   │   └── AGENT.md
│   └── api-designer/
│       └── AGENT.md
└── skills/
    ├── security-review-workflow/
    │   └── SKILL.md
    └── api-design-check/
        └── SKILL.md
```

## 相互参照

### Agent → Skill（公式パターン）

```yaml
# AGENT.md
skills:
  - secure-coding-patterns
```

### Skill → Agent（拡張パターン）

```yaml
# SKILL.md
agents:
  - name: security-auditor
    phase: review
```

## 関連

- `/gen-agents`: エージェントのみ生成
- `/gen-skills`: スキルのみ生成

## 詳細仕様

- [agent-template.md](../skills/skill-from-history/references/agent-template.md)
- [skill-template.md](../skills/skill-from-history/references/skill-template.md)
- [agent-detection.md](../skills/skill-from-history/references/agent-detection.md)
