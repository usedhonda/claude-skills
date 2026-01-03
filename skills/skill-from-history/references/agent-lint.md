# Agent Lint Rules

エージェントプロファイル（AGENT.md）のバリデーションルール。

---

## ルールカテゴリ

| カテゴリ | プレフィックス | 対象 |
|---------|---------------|------|
| Agent Core | AGNT | AGENT.md 基本構造 |
| Cross Reference | XREF | スキル-エージェント相互参照 |
| File Structure | FILE | ファイルサイズ・構造 |

---

## AGNT: Agent Core Rules

### AGNT-001: name フィールド必須

**重大度**: Error

```yaml
# NG
---
description: セキュリティレビュー専門
---

# OK
---
name: security-auditor
description: セキュリティレビュー専門
---
```

**検証**:
- `name` フィールドが存在する
- 空文字でない

### AGNT-002: name フォーマット

**重大度**: Error

```yaml
# NG
name: SecurityAuditor    # 大文字
name: security_auditor   # アンダースコア
name: 123-agent          # 数字開始

# OK
name: security-auditor
name: code-reviewer-v2
```

**検証**:
- パターン: `^[a-z][a-z0-9-]*$`
- 最大64文字

### AGNT-003: description フィールド必須

**重大度**: Error

```yaml
# NG
---
name: security-auditor
---

# OK
---
name: security-auditor
description: セキュリティ脆弱性レビュー専門
---
```

**検証**:
- `description` フィールドが存在する
- 空文字でない

### AGNT-004: description 長さ制限

**重大度**: Warning

```yaml
# NG (256文字超過)
description: [非常に長い説明...]

# OK
description: セキュリティ脆弱性レビュー専門。認証フロー、入力検証を確認
```

**検証**:
- 最大256文字
- 超過時: 自動トランケート提案

### AGNT-005: Role セクション必須

**重大度**: Warning

```markdown
# NG
# Security Auditor

## When to Use
...

# OK
# Security Auditor

## Role

認証、認可、入力検証の観点でコードをレビュー。
```

**検証**:
- `## Role` セクションが存在する

### AGNT-006: Prompt Template セクション必須

**重大度**: Warning

```markdown
# NG
## Role
...
## When to Use
...

# OK
## Role
...
## Prompt Template

```
セキュリティ監査を行います...
```
```

**検証**:
- `## Prompt Template` セクションが存在する
- コードブロックを含む

### AGNT-007: When to Use セクション必須

**重大度**: Warning

```markdown
# OK
## When to Use

このエージェントを起動する条件：
- 認証コードのレビュー時
- APIセキュリティ監査時
```

**検証**:
- `## When to Use` セクションが存在する
- リスト項目を含む

### AGNT-008: skills 参照先存在

**重大度**: Warning

```yaml
# NG (存在しないスキル)
skills:
  - nonexistent-skill

# OK
skills:
  - secure-coding-patterns  # .claude/skills/ に存在
```

**検証**:
- `skills:` の各項目が `.claude/skills/[name]/SKILL.md` として存在

---

## XREF: Cross Reference Rules

### XREF-001: agents 参照先存在

**重大度**: Error

SKILL.md 内の `agents:` 参照をチェック。

```yaml
# SKILL.md
---
agents:
  - name: security-auditor  # 存在確認
---
```

**検証**:
- `.claude/agents/[name]/AGENT.md` が存在する

### XREF-002: 循環参照禁止

**重大度**: Error

```
# NG: 循環参照
Agent A → Skill B → Agent A
```

**検証**:
- Agent → Skill → Agent の循環がない
- 依存グラフが DAG（有向非巡回グラフ）

### XREF-003: 宣言と使用の一致

**重大度**: Warning

```yaml
# SKILL.md frontmatter
agents:
  - name: security-auditor
  - name: performance-analyzer  # Warning: 本文で未使用
```

**検証**:
- frontmatter の `agents:` と本文での参照が一致

---

## FILE: File Structure Rules

### FILE-001: AGENT.md サイズ制限

**重大度**: Warning

```
# NG
AGENT.md: 350 lines  # > 300

# OK
AGENT.md: 250 lines  # < 300
```

**検証**:
- AGENT.md は300行以内
- 超過時: references/ への分離を推奨

### FILE-002: references サイズ制限

**重大度**: Info

```
# NG
references/patterns.md: 250 lines  # > 200

# OK
references/patterns.md: 150 lines  # < 200
```

**検証**:
- 各 references/*.md は200行以内

### FILE-003: ディレクトリ構造

**重大度**: Info

```
# 推奨構造
.claude/agents/[name]/
├── AGENT.md
└── references/
    └── *.md
```

**検証**:
- AGENT.md がルートに存在
- サブディレクトリは references/ のみ

---

## 重大度レベル

| レベル | 動作 | 例 |
|--------|------|---|
| Error | 生成失敗 | name 未定義、循環参照 |
| Warning | 警告表示、生成継続 | description 超過、セクション欠落 |
| Info | 情報表示のみ | ファイルサイズ推奨 |

---

## 自動修正対応

| ルール | 自動修正 | 内容 |
|--------|---------|------|
| AGNT-002 | Yes | 小文字変換、アンダースコア→ハイフン |
| AGNT-004 | Yes | 256文字でトランケート |
| FILE-001 | No | references/ 分離は手動 |

---

## Lint 実行タイミング

1. **生成時**: エージェント生成完了後に自動実行
2. **保存時**: AGENT.md 保存時にバックグラウンド実行
3. **コミット前**: pre-commit hook で実行（オプション）

---

## エラーメッセージ例

```
[AGNT-001] Error: 'name' field is required in YAML frontmatter
  at .claude/agents/my-agent/AGENT.md:1

[AGNT-005] Warning: Missing '## Role' section
  at .claude/agents/security-auditor/AGENT.md
  Suggestion: Add a Role section describing the agent's responsibilities

[XREF-001] Error: Referenced agent 'nonexistent-agent' not found
  at .claude/skills/my-skill/SKILL.md:5
  Expected: .claude/agents/nonexistent-agent/AGENT.md

[FILE-001] Warning: AGENT.md exceeds 300 lines (current: 350)
  at .claude/agents/complex-agent/AGENT.md
  Suggestion: Extract detailed content to references/ directory
```
