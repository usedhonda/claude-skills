# 調査結果: claude-skills プロジェクト構造の最適化

## 結論

**現在の構造は正しいが、2026年推奨パターンに合わせて改善可能。**

## 検証結果

| 項目 | 状態 | 備考 |
|------|------|------|
| `settings.json` + `marketplace.json` | ✅ 正しい | 両方必要。v4.5.0で同期済み |
| コマンドの配置（skills内） | ✅ 正しい | `learn-*`, `par-*` のプレフィックス |
| marketplace.json の `commands`/`agents` | ✅ 正しい | 両フィールド存在 |
| agents パスが `.md` で終わる | ✅ 正しい | `./agents/xxx/AGENT.md` 形式 |
| SKILL.md フロントマター | ✅ 正しい | name, description, compression-anchors |
| 双方向参照（Skill↔Agent） | ✅ 正しい | 両方向で参照 |

## 最新のClaude Code変更点（2026年）

### 1. コマンドとスキルの統合

```
旧: .claude/commands/review.md
新: .claude/skills/review/SKILL.md

両方とも /review として動作（後方互換）
```

**このプロジェクトへの影響**: なし。現在の構造で問題なし。

### 2. プラグイン検証コマンド

```bash
/plugin validate
```

marketplace.json と plugin.json の検証が可能に。

### 3. Agent Skills オープンスタンダード

Claude Code は [agentskills.io](https://agentskills.io) 標準に準拠。
OpenAI も同じ標準を採用。

## 軽微な改善提案（任意）

### A. `/plugin validate` を CLAUDE.md に追記

Pre-release チェックリストに追加：

```bash
# プラグイン検証
/plugin validate
```

### B. README の更新

最新のインストール方法を明記：

```bash
/plugin marketplace add usedhonda/claude-skills
/plugin install parallel-dev-orchestrator
```

---

## 2026年推奨パターン

### 推奨構造

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # メタデータのみ
├── commands/                # ルートレベルのコマンド（オプション）
├── skills/
│   └── skill-name/
│       ├── SKILL.md         # 100-200行に抑える
│       ├── references/      # 詳細ドキュメント
│       ├── templates/       # テンプレート
│       └── scripts/         # ユーティリティ
├── agents/
│   └── agent-name/
│       └── AGENT.md
└── hooks/
    └── hooks.json
```

### 重要な原則

| 原則 | 説明 |
|------|------|
| Progressive Disclosure | SKILL.mdは500行以下、詳細はreferences/へ |
| コマンド分離 | 1コマンド = 1ファイル |
| エージェント活用 | 複雑な処理はエージェントに委譲 |

---

## 現在の構造 vs 推奨構造

### 現状

```
skills/parallel-dev-orchestrator/
├── SKILL.md              # 300行+（大きい）
├── commands/             # ✅ 正しい
├── templates/            # ✅ 正しい
└── references/           # ✅ 正しい
```

### 改善案

| 項目 | 現状 | 改善 |
|------|------|------|
| SKILL.md サイズ | 300行+ | 100-200行に削減 |
| 詳細ドキュメント | SKILL.md内 | references/ に移動 |
| コマンド位置 | skills/内 | そのまま（正しい） |

---

## 具体的な改善タスク

### 1. SKILL.md のスリム化

**対象**: `skills/parallel-dev-orchestrator/SKILL.md`

移動すべきセクション:
- Task Card Format → `references/task-card-format.md`
- Dispatch Methods 詳細 → `references/dispatch-methods.md`（既存を活用）
- Merge Strategy 詳細 → `references/merge-strategy.md`

SKILL.md に残すもの:
- TL;DR
- Core Workflow（図）
- Quick Start
- 各セクションへのリンク

### 2. skill-from-history も同様

**対象**: `skills/skill-from-history/SKILL.md`

移動すべきセクション:
- 詳細なプロセス説明 → references/
- ガバナンス詳細 → references/

### 3. settings.json → plugin.json（任意）

最新の推奨は `plugin.json` だが、`settings.json` も動作する。
統一のため `plugin.json` にリネームを検討。

---

## 実装計画

### Phase 1: SKILL.md スリム化
1. parallel-dev-orchestrator/SKILL.md を分割
2. skill-from-history/SKILL.md を分割
3. 各SKILLから references/ へリンク

### Phase 2: 設定ファイル整理（任意）
1. settings.json → plugin.json にリネーム
2. marketplace.json のフォーマット確認

### Phase 3: テスト
1. `/plugin validate` で検証
2. 別プロジェクトでインストールテスト

---

## 変更ファイル

| ファイル | 変更内容 |
|----------|----------|
| `skills/parallel-dev-orchestrator/SKILL.md` | スリム化、references/へリンク |
| `skills/parallel-dev-orchestrator/references/*.md` | 詳細コンテンツ移動 |
| `skills/skill-from-history/SKILL.md` | スリム化 |
| `.claude-plugin/settings.json` | plugin.json にリネーム（任意） |
