# claude-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

## Manifesto

A project is a living argument with itself.

Every "don't do that" and every "we chose this" is part of its culture—usually lost, repeated, and paid for again.

**claude-skills** externalizes that memory: not as static doctrine, but as evidence-backed, reviewable guidance that can evolve, decay, and be forgotten.

In the agent era, collaboration isn't just about writing code—it's about making decisions reproducible.

## Principles

| Principle | Description |
|-----------|-------------|
| **Project Learning** | Learn from YOUR history, not generic best practices |
| **Negative Learning** | Learn what NOT to do from corrections |
| **Evidence-first** | Every pattern links to its source `[E1][E2]` |
| **Context Engineering** | Optimize for attention, not just tokens |
| **Rules have half-life** | Learning includes forgetting |

## How It Learns

```
Conversations + Commits + Code
         ↓
    Pattern Extraction
         ↓
    Evidence Linking
         ↓
    Human Review Gate
         ↓
    Skill Generation
         ↓
    Decay & Archival
```

## Use Cases

### "I keep making the same correction"

> "No, don't use Express. We use Hono now."

Your correction becomes a constraint. Next time, the AI already knows.

### "New members don't know our conventions"

Tribal knowledge evaporates. claude-skills extracts it from actual behavior—with receipts.

### "The AI forgot what we discussed"

Long contexts lose focus in the middle. We anchor critical decisions at the edges.

## Compared to

| Tool | Approach | claude-skills |
|------|----------|---------------|
| **Cursor Rules** | Declare rules (human writes) | Observe patterns (history extracts) |
| **Devin Knowledge** | Store knowledge | Make knowledge executable |
| **Generic linters** | Enforce universal rules | Enforce YOUR conventions |

## Available Skills

### [skill-from-history](skills/skill-from-history/SKILL.md)

> **Project-Specific Skill Generator** - The meta-skill that powers this philosophy.

**Key Features:**
- Multi-source analysis (conversations, Git, codebase)
- Negative Learning (anti-pattern detection from corrections)
- Evidence linking with `[E1][E2]` references
- Review Gate for Critical constraints
- Conflict detection and resolution
- Command generation for frequently-used skills

**v3.0 Governance:**
- Constraint Maturity (Draft → Accepted → Canonical → Deprecated)
- Epochs (time-based rule archival)
- Golden Tasks (evaluation harness)
- Cross-Project Learning (global skills with quarantine)

**Quick Start:**
```bash
/gen-all        # Generate agents + skills
/skill-eval     # Run Golden Tasks
/skill-promote  # Promote validated constraints
```

Or say: "create skills from history", "analyze patterns"

## Installation

### Via Plugin (Recommended)

```bash
# Add marketplace
/plugin marketplace add usedhonda/claude-skills

# Install
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

Issues and PRs are welcome!

## License

MIT License - see [LICENSE](LICENSE)

---

# 日本語

## マニフェスト

プロジェクトは、自分自身との対話である。

「それはやるな」「こっちを選んだ」—その一つ一つがプロジェクトの文化だ。しかし多くは失われ、繰り返され、また代償を払うことになる。

**claude-skills** はその記憶を外部化する。静的な教義としてではなく、証拠に裏付けられ、レビュー可能で、進化し、衰退し、忘れられることもできる知識として。

エージェント時代の協働とは、コードを書くことではない。意思決定を再現可能にすることだ。

## 原則

| 原則 | 説明 |
|------|------|
| **プロジェクト学習** | 汎用ベストプラクティスではなく、あなたの履歴から学ぶ |
| **負の学習** | 修正履歴から「すべきでないこと」も学ぶ |
| **証拠優先** | すべてのパターンにソースを紐付け `[E1][E2]` |
| **コンテキスト工学** | トークン量ではなく、注意の最適化 |
| **ルールには賞味期限がある** | 学習には忘却が含まれる |

## 学習の仕組み

```
会話 + コミット + コード
         ↓
    パターン抽出
         ↓
    証拠リンク
         ↓
    人間によるレビュー
         ↓
    スキル生成
         ↓
    衰退とアーカイブ
```

## ユースケース

### 「また同じ指摘をしている」

> 「違う、Expressは使わない。今はHonoを使ってる」

あなたの修正が制約になる。次からAIは最初から知っている。

### 「新メンバーが規約を知らない」

暗黙知は蒸発する。claude-skillsは実際の行動から抽出する—証拠付きで。

### 「AIが話した内容を忘れた」

長いコンテキストは中央部で集中力を失う。重要な決定を端にアンカーする。

## 他ツールとの比較

| ツール | アプローチ | claude-skills |
|--------|----------|---------------|
| **Cursor Rules** | ルールを宣言（人間が書く） | パターンを観測（履歴から抽出） |
| **Devin Knowledge** | 知識を保管 | 知識を実行可能に |
| **汎用リンター** | 普遍的ルールを適用 | あなたの規約を適用 |

## 利用可能なスキル

### [skill-from-history](skills/skill-from-history/SKILL.md)

> **プロジェクト特化型スキル生成** - この思想を実現するメタスキル。

**主な機能:**
- マルチソース分析（会話、Git、コードベース）
- 負の学習（修正履歴からのアンチパターン検出）
- 証拠リンク `[E1][E2]`
- Critical制約のレビューゲート
- 衝突検出と解決
- 頻用スキルのコマンド生成

**v3.0 ガバナンス:**
- 制約成熟度（Draft → Accepted → Canonical → Deprecated）
- Epochs（時代ベースのルールアーカイブ）
- Golden Tasks（評価ハーネス）
- クロスプロジェクト学習（検疫付きグローバルスキル）

**クイックスタート:**
```bash
/gen-all        # エージェント + スキル生成
/skill-eval     # Golden Tasks実行
/skill-promote  # 検証済み制約の昇格
```

または「履歴からスキル作成」「パターン分析」と入力

## インストール

### プラグイン経由（推奨）

```bash
# マーケットプレイス追加
/plugin marketplace add usedhonda/claude-skills

# インストール
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

Issue・PR歓迎！
