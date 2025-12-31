# Execution Process Details

Detailed criteria and tables for the skill-from-history execution steps.

## Step 4: Pattern Extraction Criteria

Identify potential skills based on weighted scoring:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Frequency | 30% | Pattern appears in 3+ sessions/commits |
| Reusability | 25% | Contains extractable code templates |
| Complexity | 20% | Multi-step, non-trivial implementation |
| Documentation | 15% | Well-explained in history |
| Specificity | 10% | Project-specific value |

## Step 4.5: Anti-pattern Detection Signals

Identify patterns the user has explicitly rejected or corrected.

**Detection Signals**:

| Signal Type | Examples | Weight |
|-------------|----------|--------|
| Direct rejection | "No", "Don't", "Never use", "That's wrong" | High |
| Correction | "Actually...", "Instead of X, use Y", "Not like that" | High |
| Deprecation | "That's outdated", "We don't use X anymore" | Medium |
| Convention violation | "We always use X for this", "Follow our pattern" | Medium |
| Re-request | Same question asked again after assistant response | Low |

**Anti-pattern Structure**:

| Field | Description |
|-------|-------------|
| trigger | What caused the wrong response |
| wrong_approach | What was rejected |
| correct_approach | What should be done instead |
| reason | Why (if stated) |
| severity | critical / warning / info |

**Severity Classification**:

| Confidence | Frequency | Severity |
|------------|-----------|----------|
| > 80% | 3+ | Critical (must avoid) |
| > 60% | 2+ | Warning (discouraged) |
| > 40% | 1+ | Info (consider avoiding) |

## Step 5: Pattern Categories

**Development & Technical**:
- Error handling approaches
- API design conventions
- State management patterns
- Data validation methods
- Testing strategies

**Workflow & Process**:
- Deployment procedures
- Code review processes
- Testing workflows
- Build and release processes

**Creative & Design**:
- UI component patterns
- Styling conventions
- Animation patterns
- Layout templates

**Document & Data**:
- Documentation templates
- Data transformation patterns
- Report generation
- Configuration management

## Step 6: Conflict Detection Details

### Similarity Check Methods

- **Name similarity**: Levenshtein distance on skill names
- **Keyword overlap**: Common terms in descriptions
- **Pattern overlap**: Similar code structures

### Skill Conflict Types

| Type | Severity | Description |
|------|----------|-------------|
| Duplicate | High | 80%+ similarity with existing skill |
| Overlap | Medium | 50-80% similarity, partial coverage |
| Missing Dependency | Medium | Pattern requires another skill |
| Outdated Reference | Low | References deprecated code/APIs |

### Semantic Conflict Detection (for constraints)

| Conflict Type | Detection | Resolution |
|---------------|-----------|------------|
| Direct contradiction | Same subject, opposite polarity | User chooses |
| Temporal evolution | Newer supersedes older | `supersede` |
| Scope overlap | Same context, different actions | Define scopes |

### Resolution Options

- **Merge**: Combine with existing skill
- **Extend**: Add as variation to existing skill
- **Replace/Supersede**: Mark old as superseded, link to new
- **Scope**: Both valid in different contexts
- **Skip**: Do not generate

## Step 7: Proposal Display

Candidates are grouped by:

- **New Skill Candidates**: Patterns worthy of new skills
- **Additions to Existing Skills**: Extensions to current skills
- **Similar to Existing Skills**: Potential duplicates requiring decision
- **Detected Constraints**: Anti-patterns from negative learning

### Command Recommendation Criteria

| Recommendation | Criteria |
|----------------|----------|
| ✓ Recommend | Clear trigger action, repeated execution, faster than natural language |
| △ Optional | Useful but not essential |
| - Skip | Reference-only patterns, high context dependency |

### Example Proposal Table

| # | Skill | Category | Confidence | Command? | Reason |
|---|-------|----------|------------|----------|--------|
| 1 | multi-ai-review | Workflow | 92% | ✓ Recommend | Clear action trigger |
| 2 | commit-convention | Workflow | 85% | △ Optional | Reference at commit time |
| 3 | doc-structure | Document | 88% | - Skip | Pattern reference only |

## Step 7.5: Review Gate

Constraints meeting Critical thresholds (confidence > 80%, frequency >= 3) are marked "Pending Critical" and require user approval:

- `approve` - Confirm as Critical (blocks generation if violated)
- `warning` - Demote to Warning (shows warning but allows)
- `skip` - Remove constraint
