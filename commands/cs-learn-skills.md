---
description: claude-skills knowledge management (init/gen/check/promote/status)
argument-hint: init|gen|check|promote|status [options]
---

# Knowledge Management

Claude Code の会話履歴からスキルとエージェントを生成・管理。

## Usage

```
/cs-learn-skills                  # ヘルプ表示
/cs-learn-skills status           # 現在状態
/cs-learn-skills init             # Golden Tasks 初期化
/cs-learn-skills gen              # agents/skills 生成
/cs-learn-skills check            # 評価実行
/cs-learn-skills promote          # 成熟度昇格
```

---

## Subcommands

### status - 現在状態表示

```
/cs-learn-skills status
```

**出力例**:

```
📊 Project Knowledge Status

Skills:
├─ Generated: 3
├─ Maturity: Draft(2) Accepted(1) Canonical(0)
└─ Last updated: 2h ago

Agents:
├─ Generated: 1
└─ worktree-dispatcher (active)

Golden Tasks:
└─ ⚠️ Not found

Next Action:
→ /cs-learn-skills init
```

---

### init - Golden Tasks 初期化

```
/cs-learn-skills init
```

Golden Tasks の雛形を作成し、評価基盤をセットアップ。

**作成されるファイル**:

```
.claude/golden-tasks/
├── index.yaml          # タスク一覧
├── GT-001.yaml         # サンプルタスク
└── README.md           # 作成ガイド
```

**Golden Task 形式**:

```yaml
id: GT-001
name: "基本的なコード生成"
input:
  context: "Express.js プロジェクト"
  request: "ユーザー認証エンドポイントを追加"
expected:
  constraints_applied: ["CONS-001", "CONS-002"]
  patterns_used: ["error-handling", "validation"]
  violations: []
```

---

### gen - エージェント/スキル生成

```
/cs-learn-skills gen                # 両方生成
/cs-learn-skills gen --agents       # エージェントのみ
/cs-learn-skills gen --skills       # スキルのみ
```

**実行プロセス**:

```
1. ソース収集
   └── 会話履歴、Git、コードベース

2. エージェントパターン抽出
   └── 役割/ペルソナの検出

3. スキルパターン抽出
   └── ワークフロー/手順の検出

4. 競合検出
   └── エージェント間、スキル間

5. 提案表示（候補 + Confidence）

6. ユーザー選択 → 生成
```

**出力先**:

```
.claude/
├── agents/
│   └── {name}/AGENT.md
└── skills/
    └── {name}/SKILL.md
```

---

### check - 評価実行

```
/cs-learn-skills check                      # 全 Golden Tasks 実行
/cs-learn-skills check --baseline           # ベースライン比較
/cs-learn-skills check --regression CONS-XXX  # 回帰テスト
/cs-learn-skills check --ci                 # CI モード
```

**出力例**:

```markdown
## Golden Task Results

| Task | Status | Duration | Notes |
|------|--------|----------|-------|
| GT-001 | PASS | 2.3s | All criteria met |
| GT-002 | FAIL | 1.8s | CONS-003 violated |

## Summary
- Passed: 1 (50%)
- Failed: 1
```

---

### promote - 成熟度昇格

```
/cs-learn-skills promote                    # 候補表示
/cs-learn-skills promote CONS-XXX           # 特定制約を昇格
/cs-learn-skills promote --all-eligible     # 一括昇格
/cs-learn-skills promote --diff CONS-XXX    # 差分プレビュー
```

**成熟度レベル**:

| レベル | 強制力 | 要件 |
|--------|--------|------|
| Draft | Warning | 初期状態 |
| Accepted | Block (回避可) | Eval >= 80% |
| Canonical | Block (固定) | Eval >= 95%, 30日安定 |

---

## Quick Start

```
1. /cs-learn-skills status    # 現在状態を確認
2. /cs-learn-skills init      # Golden Tasks 作成
3. /cs-learn-skills gen       # スキル生成
4. /cs-learn-skills check     # 評価
5. /cs-learn-skills promote   # 昇格
```

---

## 詳細ドキュメント

- [skill-template.md](../skills/skill-from-history/references/skill-template.md)
- [agent-template.md](../skills/skill-from-history/references/agent-template.md)
- [golden-tasks.md](../skills/skill-from-history/references/golden-tasks.md)
- [lifecycle.md](../skills/skill-from-history/references/lifecycle.md)
