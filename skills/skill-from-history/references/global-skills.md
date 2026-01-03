# Global Skills (Cross-Project Learning)

複数プロジェクト間でスキルを共有するための仕様。

---

## 概念

> "Cross-Project Learning: synthesize skills from multiple local projects" (Gemini)

Global Skills = 複数プロジェクトで共通して使えるスキル。
個人の「標準ライブラリ」として機能。

### セキュリティ原則

> "外部ルールは Canonical に自動昇格しない" (Gemini: Maturity Gating)

---

## ディレクトリ構造

```
~/.claude/
├── global-skills/           # グローバルスキル
│   ├── index.yaml           # スキル一覧
│   ├── logging-patterns/
│   │   └── SKILL.md
│   ├── error-handling/
│   │   └── SKILL.md
│   └── testing-conventions/
│       └── SKILL.md
├── global-agents/           # グローバルエージェント
│   └── general-reviewer/
│       └── AGENT.md
└── global-config.yaml       # グローバル設定
```

---

## スキルの昇格フロー

```
Project A                    Global                    Project B
    │                           │                           │
    ▼                           ▼                           ▼
[local skill]              [global skill]             [imported]
Canonical ────► Promote ────► Global ────► Import ────► Draft
                   │                           │
                   ▼                           ▼
              Quarantine                  Quarantine
```

### 1. Local → Global 昇格

```bash
/skill-globalize SKILL-001
```

条件:
- Local で Canonical 状態
- Eval Pass Rate >= 95%
- 3つ以上のプロジェクトで類似パターン（推奨）

### 2. Global → Local インポート

```bash
/skill-import logging-patterns
```

インポート時:
- **Maturity**: Draft（強制）
- **Flag**: `external: true`
- **Provenance**: 元プロジェクト情報を記録

---

## Provenance Labels

外部スキルの出所を追跡。

```yaml
# .claude/skills/logging-patterns/SKILL.md
---
name: logging-patterns
maturity: draft
external: true
provenance:
  source_project: my-api-project
  source_epoch: E2
  source_maturity: canonical
  imported_at: 2025-01-03
  imported_by: user@example.com
  global_skill_id: logging-patterns-v1
---
```

### Provenance の用途

1. **トレーサビリティ**: どこから来たか追跡
2. **更新通知**: 元スキルが更新されたら通知
3. **セキュリティ**: 外部ルールを識別

---

## Quarantine Lane

外部スキルは「検疫」状態で開始。

### 検疫ルール

| 項目 | 制限 |
|------|------|
| Maturity | Draft 固定（昇格は手動のみ） |
| 強制力 | Info のみ（Warning/Error なし） |
| TTL | 14日（レビューなしで自動削除提案） |
| Eval | 必須（ローカル Golden Tasks で検証） |

### 検疫解除条件

```yaml
quarantine_release:
  eval_pass_rate: 0.8
  local_reaffirmations: 2
  days_in_quarantine: 7
  user_approval: true
```

### 検疫ステータス表示

```bash
/skill-status --quarantine

## Quarantined Skills

| Skill | Source | Days | Eval | Action |
|-------|--------|------|------|--------|
| logging-patterns | my-api | 5 | 85% | [Release] |
| error-handling | shared | 2 | - | [Pending eval] |
```

---

## Secret Redaction

インポート/エクスポート時の機密情報除去。

### 検出パターン

```python
REDACTION_PATTERNS = [
    # API Keys
    r'[A-Za-z0-9_-]{20,}',  # 長いランダム文字列
    r'sk-[A-Za-z0-9]{32,}',  # OpenAI形式
    r'ghp_[A-Za-z0-9]{36}',  # GitHub PAT

    # Secrets
    r'password\s*[:=]\s*["\']?[^"\']+',
    r'secret\s*[:=]\s*["\']?[^"\']+',
    r'token\s*[:=]\s*["\']?[^"\']+',

    # URLs with credentials
    r'https?://[^:]+:[^@]+@',

    # Private paths
    r'/Users/[^/]+/',
    r'/home/[^/]+/',
    r'C:\\Users\\[^\\]+\\',
]
```

### 除去プロセス

```python
def redact_for_export(skill):
    content = skill.content

    for pattern in REDACTION_PATTERNS:
        content = re.sub(pattern, '[REDACTED]', content)

    # Evidence の excerpt も処理
    for evidence in skill.evidence:
        evidence.excerpt = redact(evidence.excerpt)

    return content
```

### 除去レポート

```
## Redaction Report

Skill: logging-patterns

Redacted items:
- Line 45: API key pattern → [REDACTED]
- Line 78: Private path → [REDACTED]
- Evidence E3: Token pattern → [REDACTED]

Total: 3 items redacted
```

---

## Global Config

```yaml
# ~/.claude/global-config.yaml

global_skills:
  enabled: true
  auto_import: false  # 自動インポートは無効
  quarantine_days: 14
  redaction:
    enabled: true
    patterns_file: ~/.claude/redaction-patterns.yaml

sync:
  enabled: false  # クラウド同期（将来機能）
  endpoint: null

allowed_sources:
  - my-api-project
  - shared-utils
  - team-standards

blocked_patterns:
  - "*.credentials*"
  - "*secret*"
```

---

## コマンド

### エクスポート（Local → Global）

```bash
/skill-globalize SKILL-001
```

オプション:
- `--dry-run`: 実際には保存しない
- `--redact`: 機密情報を除去（デフォルト有効）
- `--force`: 確認なしで実行

### インポート（Global → Local）

```bash
/skill-import logging-patterns
```

オプション:
- `--list`: 利用可能なグローバルスキル一覧
- `--preview`: インポート内容をプレビュー
- `--skip-quarantine`: 検疫をスキップ（非推奨）

### 管理

```bash
/skill-global --list           # グローバルスキル一覧
/skill-global --update         # 更新チェック
/skill-global --cleanup        # 未使用スキル削除
```

---

## 更新通知

元スキルが更新された場合:

```markdown
## Global Skill Update Available

**Skill**: logging-patterns
**Current**: v1.0 (imported 2025-01-03)
**Available**: v1.1 (updated 2025-01-10)

**Changes**:
- Added: New log format pattern
- Updated: Evidence index (+2 entries)

Actions:
- [Update] Import new version (quarantine applies)
- [Ignore] Keep current version
- [Pin] Never notify for this skill
```

---

## 統合例

### 個人ワークフロー

1. Project A で `logging-patterns` を開発
2. Canonical に昇格後、`/skill-globalize`
3. Project B で `/skill-import logging-patterns`
4. 14日間の検疫期間で評価
5. 問題なければ Accepted に昇格

### チームワークフロー

```yaml
# team-standards リポジトリ
.claude/
├── skills/
│   └── team-conventions/
│       └── SKILL.md
└── export-config.yaml
```

```yaml
# export-config.yaml
export:
  skills:
    - team-conventions
  redaction: strict
  versioning: semantic
```

チームメンバーは:
```bash
/skill-import --from team-standards team-conventions
```

---

## セキュリティ考慮事項

### リスク

1. **Poisoning**: 悪意あるスキルのインポート
2. **Leakage**: 機密情報のエクスポート
3. **Drift**: 元スキルとの乖離

### 対策

| リスク | 対策 |
|--------|------|
| Poisoning | Quarantine + Local Eval + User Approval |
| Leakage | Redaction + Allowlist + Review |
| Drift | Provenance + Update Notification |

### 監査ログ

```yaml
# ~/.claude/global-audit.log
- timestamp: 2025-01-03T10:00:00Z
  action: export
  skill: logging-patterns
  source: my-api-project
  redactions: 3

- timestamp: 2025-01-03T11:00:00Z
  action: import
  skill: logging-patterns
  target: new-project
  quarantine: true
```
