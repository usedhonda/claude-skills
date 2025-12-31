# claude-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

> A collection of Claude Code skills for enhanced development workflows

## Available Skills

### [skill-from-history](skills/skill-from-history/SKILL.md)

> **Project-Specific Skill Generator** - Learn from YOUR project's history, not generic best practices.

#### Why Project-Specific?

| Generic Tools | skill-from-history |
|---------------|-------------------|
| Analyze current code only | Analyze **YOUR** conversations + commits + code |
| Generic best practices | **YOUR** team's actual conventions |
| One-size-fits-all rules | Tailored to **YOUR** project |
| Learn "what to do" | Learn "what to do" + **"what NOT to do"** |

#### Key Features

- **Multi-source analysis**: Conversation history, Git commits, and codebase
- **Negative Learning**: Detects corrections you made to prevent repeated mistakes
- **Evidence Linking**: Every pattern links to its source evidence `[E1][E2]`
- **Review Gate**: Human approval required before Critical constraints
- **Skill Lint**: Auto-validate generated skills for quality
- **4 pattern categories**: Development, Workflow, Creative, Document
- **Conflict detection**: Prevents duplicate skills

#### Negative Learning (Unique)

Most tools learn "what to do". This skill also learns **"what NOT to do"**:

```
User: "No, don't use Express. We use Hono now."
         ↓
   [Detected as anti-pattern]
         ↓
   Constraint added to skills:
   "When creating API endpoints, use Hono framework (not Express)"
```

#### Quick Start

```bash
/skill-from-history
```

Or say: "create skills from history", "analyze patterns"

## Installation

### Via Plugin (Recommended)

```bash
# Add marketplace
/plugin marketplace add usedhonda/claude-skills

# Install specific skill
/plugin install skill-from-history@usedhonda-claude-skills
```

### Manual Installation

```bash
# Global (all projects)
mkdir -p ~/.claude/skills/skill-from-history
cp -r skills/skill-from-history/* ~/.claude/skills/skill-from-history/

# Project-specific
mkdir -p .claude/skills/skill-from-history
cp -r skills/skill-from-history/* .claude/skills/skill-from-history/
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) (Pro, Max, Team, or Enterprise)
- Git repository (recommended for full analysis)

## Contributing

Issues and PRs are welcome! To add a new skill:

1. Create `skills/[skill-name]/SKILL.md`
2. Add `references/` directory if needed
3. Update this README

## License

MIT License - see [LICENSE](LICENSE)

---

## 日本語

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

> Claude Code用スキルコレクション

## 利用可能なスキル

### [skill-from-history](skills/skill-from-history/SKILL.md)

> **プロジェクト特化型スキル生成** - 汎用的なベストプラクティスではなく、あなたのプロジェクトの履歴から学習

#### なぜ「プロジェクト特化」？

| 一般的なツール | skill-from-history |
|--------------|-------------------|
| 現在のコードのみ分析 | **あなたの**会話 + コミット + コードを分析 |
| 汎用的なベストプラクティス | **あなたのチーム**の実際の規約 |
| 画一的なルール | **あなたのプロジェクト**に最適化 |
| 「すべきこと」を学習 | 「すべきこと」+ **「すべきでないこと」**を学習 |

#### 主な機能

- **マルチソース分析**: 会話履歴、Git、コードベース
- **負の学習**: 修正履歴から同じ失敗を防止
- **エビデンスリンク**: 各パターンをソースに紐付け `[E1][E2]`
- **レビューゲート**: Critical制約には人間の承認が必要
- **スキルLint**: 生成されたスキルの品質を自動検証
- **4カテゴリ分類**: Development, Workflow, Creative, Document
- **重複検出**: 既存スキルとの競合を防止

#### 負の学習（独自機能）

多くのツールは「何をすべきか」を学びます。このスキルは**「何をすべきでないか」**も学習：

```
ユーザー: 「違う、Expressは使わないで。今はHonoを使ってる」
              ↓
        [アンチパターンとして検出]
              ↓
        スキルに制約を追加:
        「APIエンドポイント作成時はHonoを使用（Expressは禁止）」
```

#### クイックスタート

```bash
/skill-from-history
```

または「履歴からスキル作成」「パターン分析」と入力

## インストール

### プラグイン経由（推奨）

```bash
# マーケットプレイス追加
/plugin marketplace add usedhonda/claude-skills

# スキルをインストール
/plugin install skill-from-history@usedhonda-claude-skills
```

### 手動インストール

```bash
# グローバル（全プロジェクト）
mkdir -p ~/.claude/skills/skill-from-history
cp -r skills/skill-from-history/* ~/.claude/skills/skill-from-history/

# プロジェクト固有
mkdir -p .claude/skills/skill-from-history
cp -r skills/skill-from-history/* .claude/skills/skill-from-history/
```

## 必要条件

- Claude Code（Pro, Max, Team, Enterprise）
- Gitリポジトリ（推奨）

## コントリビュート

Issue・PR歓迎！新しいスキルを追加するには:

1. `skills/[skill-name]/SKILL.md` を作成
2. 必要に応じて `references/` ディレクトリを追加
3. このREADMEを更新
