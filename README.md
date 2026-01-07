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

> **Project-Specific Skill Generator** - Analyze your conversation history, Git commits, and codebase to generate project-specific agents and skills.

**What it does:**
1. Reads your Claude Code conversation history
2. Analyzes Git commit patterns
3. Extracts recurring workflows and corrections
4. Generates reusable agents (roles) and skills (workflows)

**Commands:**
```bash
/skill-from-history:learn-status   # Check current learning state
/skill-from-history:learn-init     # Initialize Golden Tasks harness
/skill-from-history:learn-gen      # Generate agents/skills from history
/skill-from-history:learn-check    # Run evaluation against Golden Tasks
/skill-from-history:learn-promote  # Promote validated constraints
```

**Example workflow:**
```bash
# 1. Check what patterns exist
/skill-from-history:learn-status

# 2. Generate skills from your history
/skill-from-history:learn-gen --skills

# 3. Output: .claude/skills/{name}/SKILL.md
```

Or say: "履歴からスキル作成", "analyze patterns", "learn from history"

---

### [parallel-dev-orchestrator](skills/parallel-dev-orchestrator/SKILL.md)

> **Parallel Development Workflow** - Decompose large tasks into parallel Claude Code sessions with auto-merge.

**What it does:**
1. Decomposes an epic into isolated tasks (T01, T02, ...)
2. Creates branches and worktrees for each task
3. Dispatches tasks to multiple Claude Code sessions
4. Auto-merges PRs based on risk policy

**Commands:**
```bash
/parallel-dev-orchestrator:par-status    # Show active plans and PR status
/parallel-dev-orchestrator:par-init      # Check GitHub prerequisites
/parallel-dev-orchestrator:par-plan      # Decompose epic → plan.yaml
/parallel-dev-orchestrator:par-dispatch  # Submit tasks to sessions
/parallel-dev-orchestrator:par-harvest   # Merge PRs and generate report
```

**Example workflow:**
```bash
# 1. Check environment is ready
/parallel-dev-orchestrator:par-init

# 2. Plan task decomposition
/parallel-dev-orchestrator:par-plan "Add OAuth2 authentication"

# 3. Dispatch to parallel sessions
/parallel-dev-orchestrator:par-dispatch

# 4. Watch and merge PRs
/parallel-dev-orchestrator:par-harvest --watch
```

**Risk Policy:**
| Level | Auto-merge | When |
|-------|-----------|------|
| low | ✅ | Isolated changes, CI passing |
| medium | ✅ | + Required checks + scope isolation |
| high | ❌ | Auth, payments, data deletion → manual review |

Or say: "並列開発", "parallel tasks", "task decomposition"

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

> **プロジェクト特化型スキル生成** - 会話履歴・Git・コードベースを分析し、プロジェクト固有のスキルを自動生成。

**できること:**
1. Claude Code会話履歴を読み込み
2. Gitコミットパターンを分析
3. 繰り返しのワークフローや修正を抽出
4. 再利用可能なエージェント（役割）とスキル（手順）を生成

**コマンド:**
```bash
/skill-from-history:learn-status   # 現在の学習状態を確認
/skill-from-history:learn-init     # Golden Tasksハーネスを初期化
/skill-from-history:learn-gen      # 履歴からスキル生成
/skill-from-history:learn-check    # Golden Tasks評価実行
/skill-from-history:learn-promote  # 検証済み制約を昇格
```

**使い方の例:**
```bash
# 1. どんなパターンがあるか確認
/skill-from-history:learn-status

# 2. 履歴からスキルを生成
/skill-from-history:learn-gen --skills

# 3. 出力: .claude/skills/{name}/SKILL.md
```

または「履歴からスキル作成」「パターン分析」と入力

---

### [parallel-dev-orchestrator](skills/parallel-dev-orchestrator/SKILL.md)

> **並列開発ワークフロー** - 大きなタスクを分解し、複数のClaude Codeセッションで並列実行→自動マージ。

**できること:**
1. Epicを独立したタスク（T01, T02, ...）に分解
2. 各タスク用のブランチとworktreeを作成
3. 複数のClaude Codeセッションにタスクを投入
4. リスクポリシーに基づいてPRを自動マージ

**コマンド:**
```bash
/parallel-dev-orchestrator:par-status    # アクティブなプラン・PR状況を表示
/parallel-dev-orchestrator:par-init      # GitHub前提条件をチェック
/parallel-dev-orchestrator:par-plan      # Epic → plan.yaml に分解
/parallel-dev-orchestrator:par-dispatch  # セッションにタスク投入
/parallel-dev-orchestrator:par-harvest   # PRをマージしレポート生成
```

**使い方の例:**
```bash
# 1. 環境が準備できているか確認
/parallel-dev-orchestrator:par-init

# 2. タスク分解を計画
/parallel-dev-orchestrator:par-plan "OAuth2認証を追加"

# 3. 並列セッションに投入
/parallel-dev-orchestrator:par-dispatch

# 4. PRを監視・マージ
/parallel-dev-orchestrator:par-harvest --watch
```

**リスクポリシー:**
| レベル | 自動マージ | 条件 |
|--------|-----------|------|
| low | ✅ | 影響範囲が限定的、CI通過 |
| medium | ✅ | + 必須チェック + スコープ分離 |
| high | ❌ | 認証・決済・データ削除 → 手動レビュー必須 |

または「並列開発」「タスク分解」と入力

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
