# claude-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

A collection of Claude Code skills for enhanced development workflows.

## Why This Tool?

Unlike generic rule generators that only analyze your current codebase, **skill-from-history learns from YOUR actual development process**:

- **Your conversations** - What you repeatedly ask Claude to do
- **Your corrections** - Mistakes Claude made that you fixed
- **Your codebase** - Project-specific conventions

### Negative Learning (Unique Feature)

Most tools learn "what to do". skill-from-history also learns **"what NOT to do"**:

```
User: "No, don't use Express. We use Hono now."
         ↓
   [Detected as anti-pattern]
         ↓
   Constraint added to skills:
   "When creating API endpoints, use Hono framework (not Express)"
```

This prevents Claude from repeating the same mistakes across sessions.

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
/plugin install skill-from-history@usedhonda-claude-skills
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

### なぜこのツール？

一般的なルール生成ツールは「現在のコード」しか見ません。**skill-from-historyは「あなたの開発プロセス」から学習します**：

- **会話履歴** - Claudeに繰り返し依頼していること
- **修正履歴** - Claudeの間違いをあなたが直した記録
- **コードベース** - プロジェクト固有の規約

### 負の学習（独自機能）

多くのツールは「何をすべきか」を学びます。skill-from-historyは**「何をすべきでないか」**も学習します：

```
ユーザー: 「違う、Expressは使わないで。今はHonoを使ってる」
              ↓
        [アンチパターンとして検出]
              ↓
        スキルに制約を追加:
        「APIエンドポイント作成時はHonoを使用（Expressは禁止）」
```

これにより、セッションをまたいで同じ失敗を繰り返すことを防ぎます。

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
/plugin install skill-from-history@usedhonda-claude-skills

# 手動
mkdir -p ~/.claude/skills/skill-from-history
cp -r skills/skill-from-history/* ~/.claude/skills/skill-from-history/
```

### 使い方

```bash
/skill-from-history
```

または「履歴からスキル作成」「パターン分析」と入力
