---
description: 会話履歴からエージェントプロファイルを生成
argument-hint: [options]
---

# Agent Profile Generator

Claude Code の会話履歴を分析し、繰り返し登場する役割/ペルソナをエージェントプロファイルとして抽出・生成。

## 実行プロセス

1. **ソース収集**: 会話履歴・Git・コードベース
2. **パターン検出**: 役割記述・視点・ツールパターンを抽出
3. **候補提案**: 信頼度順にエージェント候補を表示
4. **ユーザー選択**: 生成するエージェントを選択
5. **生成・検証**: `.claude/agents/[name]/AGENT.md` を作成

## 検出シグナル

| シグナル | 重み | 例 |
|---------|------|---|
| 役割記述 | 35% | 「セキュリティ専門家として」 |
| 視点一貫性 | 30% | 繰り返しの分析観点 |
| ツールパターン | 20% | Grep + Read の組み合わせ |
| Task呼び出し | 15% | 類似プロンプトでの繰り返し |

## 出力

```
.claude/agents/[agent-name]/
├── AGENT.md           # エージェント定義
└── references/        # 詳細コンテンツ（必要時）
```

## 関連

- `/gen-skills`: スキルのみ生成
- `/gen-all`: エージェント + スキルを生成

## 詳細仕様

- [agent-template.md](../skills/skill-from-history/references/agent-template.md)
- [agent-detection.md](../skills/skill-from-history/references/agent-detection.md)
- [agent-lint.md](../skills/skill-from-history/references/agent-lint.md)
