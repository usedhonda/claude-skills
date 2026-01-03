# Agent Pattern Detection

会話履歴・Gitコミット・コードベースからエージェントパターンを検出するロジック。

---

## 検出対象

エージェントパターン = 繰り返し登場する**役割/ペルソナ**と**視点/観点**の組み合わせ。

| 要素 | 説明 | 例 |
|------|------|---|
| 役割 | 実行者の立場 | 「セキュリティ専門家として」「パフォーマンスエンジニアとして」 |
| 視点 | 分析の観点 | 「攻撃者の視点で」「ユーザー体験の観点から」 |
| ツール | 使用ツールパターン | Grep + Read の組み合わせ |
| プロンプト | 繰り返し使用されるプロンプト構造 | 「〜を分析して、〜の観点で」 |

---

## 検出シグナル

### 1. 役割記述パターン（Weight: High）

**検出キーワード**:

```
- "〜として分析"
- "〜の観点で"
- "〜の専門家として"
- "〜の立場から"
- "〜のように振る舞って"
- "as a [role]"
- "from the perspective of"
- "acting as"
```

**例**:
```
"セキュリティ専門家として、このコードをレビューして"
→ Role: security-expert
```

### 2. 視点一貫性パターン（Weight: High）

同じ視点での複数回のレビュー/分析を検出。

**検出方法**:
1. セッション間で類似の視点記述を抽出
2. 意味的類似度でクラスタリング
3. 3回以上の出現でパターン認定

**例**:
```
Session 1: "パフォーマンスの観点でチェック"
Session 2: "速度面での問題を確認"
Session 3: "レイテンシの観点から分析"
→ Perspective: performance-focused
```

### 3. ツール使用パターン（Weight: Medium）

特定のツール組み合わせが繰り返される場合。

**検出方法**:
1. tool_use ブロックのシーケンスを抽出
2. ツール組み合わせの頻度をカウント
3. 特定の役割と関連付け

**パターン例**:

| ツール組み合わせ | 推定役割 |
|----------------|---------|
| Grep → Read → Grep | code-analyzer |
| Read → Edit → Bash(test) | test-driven-developer |
| Glob → Read(multiple) | codebase-explorer |

### 4. Task ツール呼び出しパターン（Weight: Medium）

類似プロンプトでの Task ツール繰り返し起動。

**検出方法**:
```python
# JSONL から Task 呼び出しを抽出
for entry in jsonl:
    if entry.type == "tool_use" and entry.name == "Task":
        prompts.append(entry.input.prompt)

# プロンプトの類似度クラスタリング
clusters = cluster_by_similarity(prompts, threshold=0.7)

# 3回以上のクラスタをパターン認定
for cluster in clusters:
    if len(cluster) >= 3:
        agent_candidates.append(extract_agent_pattern(cluster))
```

---

## スコアリング

### 重み付け

| シグナル | 重み | 説明 |
|---------|------|------|
| 役割記述 | 35% | 明示的な役割指定 |
| 視点一貫性 | 30% | 繰り返しの視点 |
| ツールパターン | 20% | ツール使用の一貫性 |
| Taskパターン | 15% | Task呼び出しの類似性 |

### 信頼度計算

```
confidence = (
    role_score * 0.35 +
    perspective_score * 0.30 +
    tool_score * 0.20 +
    task_score * 0.15
)
```

### 閾値

| 信頼度 | 判定 |
|--------|------|
| >= 80% | 強い候補（自動提案） |
| 60-79% | 候補（ユーザー確認） |
| < 60% | 不採用 |

---

## 抽出プロセス

### Step 1: ソース収集

```
1. 会話履歴 (~/.claude/projects/[encoded-path]/)
   - agent-[agentId].jsonl
   - [sessionId].jsonl

2. Git履歴 (直近100コミット)
   - コミットメッセージ
   - 変更ファイル

3. コードベース
   - コメント内の役割記述
   - テスト名のパターン
```

### Step 2: パターン抽出

```python
def extract_agent_patterns(sources):
    patterns = []

    # 1. 役割キーワード検出
    roles = detect_role_keywords(sources.conversations)

    # 2. 視点クラスタリング
    perspectives = cluster_perspectives(sources.conversations)

    # 3. ツールパターン分析
    tool_patterns = analyze_tool_sequences(sources.conversations)

    # 4. Task呼び出し分析
    task_patterns = analyze_task_invocations(sources.conversations)

    # 5. 統合・スコアリング
    for role in roles:
        pattern = AgentPattern(
            name=suggest_name(role),
            role=role,
            perspectives=find_matching_perspectives(role, perspectives),
            tools=find_matching_tools(role, tool_patterns),
            tasks=find_matching_tasks(role, task_patterns)
        )
        pattern.confidence = calculate_confidence(pattern)
        patterns.append(pattern)

    return filter(lambda p: p.confidence >= 0.6, patterns)
```

### Step 3: 重複検出 & 更新判定

既存エージェントとの類似度チェックと更新提案。

```
既存との類似度チェック
    │
    ├─ >= 80%（重複）
    │   ├─ 新パターンあり → 更新提案
    │   └─ 新パターンなし → スキップ
    │
    ├─ 50-79%（類似）→ 拡張提案（別名で新規作成）
    │
    └─ < 50%（新規）→ 新規生成
```

### 新パターン検出

既存エージェントと新規候補を比較し、差分を抽出：

| 比較項目 | 検出方法 | 更新内容 |
|---------|---------|---------|
| エビデンス | 新規セッション/コミット | Evidence Index に追加 |
| 視点 | 新しい perspective キーワード | Role/Perspective に追記 |
| 制約 | 新しい rejection パターン | Constraints に追加 |
| ツール | 新しいツール組み合わせ | tool_patterns 更新 |
| スキル参照 | 新しいスキル使用 | skills: フィールドに追加 |

### 更新提案表示

```markdown
## Existing Agent Updates

| Agent | Match | New Evidence | New Constraints | Action |
|-------|-------|--------------|-----------------|--------|
| security-auditor | 85% | +3 sessions | +1 constraint | Update? |
| code-reviewer | 92% | +1 commit | (none) | Update? |
| api-designer | 88% | (none) | (none) | Skip |

[U1] Update security-auditor? (y/n/diff):
[U2] Update code-reviewer? (y/n/diff):
```

### 更新マージ処理

```python
def merge_agent_updates(existing: Agent, new_patterns: Patterns) -> Agent:
    # 1. Evidence Index に新規エビデンスを追加
    existing.evidence.extend(new_patterns.evidence)

    # 2. Constraints に新規制約を追加
    for constraint in new_patterns.constraints:
        if constraint not in existing.constraints:
            existing.constraints.append(constraint)

    # 3. skills: に新規参照を追加
    for skill in new_patterns.skills:
        if skill not in existing.skills:
            existing.skills.append(skill)

    # 4. 更新日時を記録
    existing.updated_at = now()

    return existing
```

### 差分表示（diff オプション）

```diff
## Evidence Index

| ID | Source | Date | Description |
|----|--------|------|-------------|
  [E1] | session:abc123 | 2024-12-15 | Original evidence |
+ [E4] | session:xyz789 | 2025-01-02 | New security pattern |
+ [E5] | commit:a1b2c3d | 2025-01-03 | Security fix commit |

## Constraints

| Pattern | Instead | Reason | Evidence |
|---------|---------|--------|----------|
  SQL string concat | Parameterized query | Injection risk | [E1] |
+ eval() usage | Safe alternatives | Code injection | [E4][E5] |
```

---

## 出力形式

### エージェント候補

```json
{
  "name": "security-auditor",
  "confidence": 85,
  "frequency": 6,
  "role": {
    "description": "セキュリティ脆弱性を分析",
    "keywords": ["security", "vulnerability", "attack"]
  },
  "perspectives": [
    "攻撃者の視点",
    "防御的コーディング"
  ],
  "tool_patterns": [
    ["Grep", "Read"],
    ["Glob", "Read"]
  ],
  "prompt_template": "セキュリティの観点で{{target}}を分析...",
  "evidence": [
    { "type": "session", "id": "abc123", "excerpt": "..." },
    { "type": "session", "id": "def456", "excerpt": "..." }
  ]
}
```

### 提案表示

```markdown
## Agent Candidates

| # | Name | Confidence | Frequency | Reason |
|---|------|------------|-----------|--------|
| A1 | security-auditor | 85% | 6 sessions | Repeated security review requests |
| A2 | performance-analyzer | 72% | 4 sessions | Performance-focused analysis |
| A3 | api-designer | 65% | 3 sessions | API design discussions |

Select agents to generate (e.g., "A1, A3"):
```

---

## 競合検出

### エージェント間競合

| 競合タイプ | 検出方法 | 解決 |
|-----------|---------|------|
| 名前重複 | 完全一致 | エラー |
| 役割重複 | 役割記述 >= 80% 類似 | マージ提案 |
| 視点重複 | 視点 >= 70% 類似 | スコープ分離 |

### エージェント-スキル競合

```
パターン: "security-review" がエージェントとスキル両方で検出

解決オプション:
1. エージェントとして生成（役割ベース）
2. スキルとして生成（手順ベース）
3. 両方生成（エージェントがスキルを使用）
```

---

## エビデンスリンク

検出したパターンをソースに紐付け。

```markdown
## Evidence Index

### Sessions
| ID | Session | Date | Excerpt |
|----|---------|------|---------|
| E1 | abc123 | 2024-12-15 | 「セキュリティ専門家として...」 |
| E2 | def456 | 2024-12-18 | 「攻撃者の視点で分析」 |

### Tool Patterns
| ID | Sequence | Count | Context |
|----|----------|-------|---------|
| E3 | Grep→Read→Grep | 5 | Security file scanning |
```
