# Pattern Detection Reference

Detailed algorithms and criteria for identifying skill-worthy patterns from project history.

## Keyword Categories

### Domain Keywords

| Category | Keywords |
|----------|----------|
| Frontend | React, Vue, Angular, component, hook, state, props, CSS, styling |
| Backend | API, REST, GraphQL, endpoint, middleware, controller, route |
| Database | query, schema, migration, model, ORM, SQL, NoSQL, Firestore |
| Authentication | auth, login, logout, token, JWT, session, OAuth, SSO |
| Deployment | deploy, build, CI/CD, Docker, Kubernetes, serverless |
| Testing | test, spec, mock, stub, fixture, assertion, coverage |
| Error Handling | error, exception, try, catch, throw, fallback, retry |
| Performance | cache, optimize, lazy, memo, debounce, throttle |

### Action Keywords

| Category | Keywords |
|----------|----------|
| Create | create, add, new, init, setup, generate, build |
| Update | update, modify, change, edit, refactor, improve |
| Delete | delete, remove, clean, purge, clear |
| Fix | fix, resolve, repair, patch, hotfix, bugfix |
| Configure | config, setting, option, parameter, env |

## JSONL Parsing

### Message Structure

```json
{
  "type": "user|assistant",
  "message": {
    "role": "user|assistant",
    "content": "string or array"
  },
  "timestamp": "ISO 8601",
  "sessionId": "UUID",
  "agentId": "string"
}
```

### Tool Call Extraction

Look for content blocks with type `tool_use`:

```json
{
  "type": "tool_use",
  "id": "tool-id",
  "name": "Edit|Write|Bash|...",
  "input": { ... }
}
```

### Code Block Extraction

From assistant messages, extract fenced code blocks:

```
```language
code content
```
```

Track:
- Language distribution
- Common patterns in code
- File paths mentioned

## Scoring Algorithm

### Frequency Score (30%)

```
frequency_score = min(occurrences / 5, 1.0)
```

- 1 occurrence: 0.2
- 3 occurrences: 0.6
- 5+ occurrences: 1.0

### Reusability Score (25%)

Evaluate based on:

| Factor | Score |
|--------|-------|
| Contains code template | +0.3 |
| Uses variables/parameters | +0.2 |
| Has clear structure | +0.2 |
| Language-agnostic concepts | +0.15 |
| Documented steps | +0.15 |

### Complexity Score (20%)

```
complexity_score = min(steps / 5, 1.0)
```

- 1 step: 0.2
- 3 steps: 0.6
- 5+ steps: 1.0

Also consider:
- Multiple tool calls: +0.1 per tool type
- File dependencies: +0.1 per file
- Error handling present: +0.2

### Documentation Score (15%)

Based on conversation context:

| Factor | Score |
|--------|-------|
| Explanation in prompts | +0.3 |
| Comments in code | +0.3 |
| Clear step descriptions | +0.2 |
| Examples provided | +0.2 |

### Specificity Score (10%)

Project-specific value:

| Factor | Score |
|--------|-------|
| Uses project-specific APIs | +0.4 |
| References project files | +0.3 |
| Follows project conventions | +0.3 |

### Total Score

```
total = (
  frequency * 0.30 +
  reusability * 0.25 +
  complexity * 0.20 +
  documentation * 0.15 +
  specificity * 0.10
)

confidence = total * 100
```

## Pattern Classification

### Coding Pattern Indicators

A pattern is classified as **coding pattern** if:

- Contains code templates or snippets
- Focuses on implementation details
- Addresses how to write specific code
- Examples: error handling, API design, validation

### Workflow Pattern Indicators

A pattern is classified as **workflow pattern** if:

- Involves multiple tools in sequence
- Describes a process or procedure
- Focuses on what to do, not just how to code
- Examples: deployment, testing, review process

### Classification Logic

```
if has_code_template AND focuses_on_implementation:
    type = "coding_pattern"
elif involves_tool_sequence OR describes_process:
    type = "workflow_pattern"
else:
    type = "hybrid"
```

## Similarity Detection

### Name Similarity

Using Levenshtein distance:

```
similarity = 1 - (levenshtein(name1, name2) / max(len(name1), len(name2)))
```

Threshold: 0.7 (70% similar)

### Keyword Overlap

Extract keywords from descriptions:

```
keywords1 = extract_keywords(desc1)
keywords2 = extract_keywords(desc2)
overlap = len(keywords1 & keywords2) / len(keywords1 | keywords2)
```

Threshold: 0.5 (50% overlap)

### Pattern Structure Similarity

Compare:
- Number of steps
- Tool types used
- File types involved
- Code structure

```
structure_sim = (
  step_similarity * 0.3 +
  tool_similarity * 0.3 +
  file_similarity * 0.2 +
  code_similarity * 0.2
)
```

Threshold: 0.6 (60% similar)

### Overall Similarity

```
overall = max(name_sim, keyword_sim, structure_sim)

if overall > 0.7:
    status = "highly_similar"
elif overall > 0.5:
    status = "somewhat_similar"
else:
    status = "distinct"
```

## Git Analysis

### Commit Message Parsing

Extract structured information:

```
Pattern: type(scope): description

Examples:
- feat(auth): add login validation
- fix(api): handle timeout errors
- refactor(db): optimize queries
```

Group commits by:
- Type (feat, fix, refactor, etc.)
- Scope (component/area)
- Keywords in description

### Change Frequency Analysis

```bash
git log --format="%H" --follow -- "path/to/file" | wc -l
```

Files with high change frequency often contain important patterns.

### Diff Pattern Analysis

Look for recurring changes:
- Same functions modified together
- Similar code additions
- Common import patterns

## Exclusion Rules

### Patterns to Exclude

| Pattern | Reason |
|---------|--------|
| One-time fixes | Not reusable |
| Environment-specific setup | Not portable |
| Generated code changes | Not intentional patterns |
| Dependency updates | Automated changes |
| Formatting-only changes | No substantive pattern |

### Minimum Requirements

To be proposed, a pattern must:
- Appear 3+ times
- Have confidence > 60%
- Not be excluded by rules above
- Not be 70%+ similar to existing skill

## Output Format

### Candidate Structure

```json
{
  "name": "suggested-skill-name",
  "type": "coding_pattern|workflow_pattern",
  "confidence": 85,
  "frequency": 5,
  "sources": [
    {"type": "session", "id": "abc123", "description": "..."},
    {"type": "commit", "hash": "def456", "message": "..."}
  ],
  "patterns": [
    {
      "description": "Main pattern description",
      "code": "code template if applicable",
      "steps": ["step1", "step2"]
    }
  ],
  "similar_to": null | {
    "skill": "existing-skill-name",
    "similarity": 0.75
  }
}
```

### Grouping for Presentation

1. **New skills**: No similarity to existing
2. **Updates**: Similar patterns found, can extend existing skill
3. **Conflicts**: High similarity, needs user decision

## Performance Considerations

### Large History Handling

For projects with extensive history:

1. **Sampling**: Analyze every Nth session
2. **Recency bias**: Weight recent sessions higher
3. **Chunking**: Process in batches of 50 sessions

### Caching

Store intermediate results:
- Parsed JSONL summaries
- Extracted patterns
- Similarity matrices

### Timeout Handling

Set limits:
- Max 100 sessions analyzed
- Max 500 commits scanned
- 30 second timeout per source

## Anti-pattern Detection (Negative Learning)

In addition to positive patterns, detect patterns that users have explicitly rejected.

### Rejection Pattern Extraction

Look for user messages following assistant responses:

**Rejection Indicators**:
- Starts with: "No", "Don't", "Wrong", "Actually", "Not"
- Contains: "instead", "rather", "should be", "we use"
- Sentiment: Negative following assistant code/suggestion

**Extraction Logic**:

```
for each (assistant_msg, user_response) pair:
  if is_rejection(user_response):
    extract_anti_pattern(
      context=assistant_msg,
      rejection=user_response,
      correction=find_correction(user_response)
    )
```

### Rejection Signal Score

| Signal | Score |
|--------|-------|
| Explicit "don't"/"never" | 1.0 |
| Correction with alternative | 0.9 |
| Preference statement | 0.7 |
| Re-request after response | 0.6 |
| Negative sentiment only | 0.4 |

### Anti-pattern Confidence

```
confidence = (
  rejection_signal * 0.40 +
  frequency * 0.30 +
  recency * 0.20 +
  explicitness * 0.10
)
```

### Severity Classification

| Confidence | Frequency | Severity |
|------------|-----------|----------|
| > 80% | 3+ | Critical |
| > 60% | 2+ | Warning |
| > 40% | 1+ | Info |

### Anti-pattern Output Structure

```json
{
  "type": "anti_pattern",
  "severity": "critical|warning|info",
  "confidence": 85,
  "pattern": {
    "trigger": "When doing X",
    "wrong": "Don't use Y",
    "correct": "Use Z instead",
    "reason": "Because..."
  },
  "sources": [
    {"session": "abc123", "excerpt": "..."}
  ]
}
```

For detailed anti-pattern detection algorithms, see [anti-pattern-detection.md](anti-pattern-detection.md)
