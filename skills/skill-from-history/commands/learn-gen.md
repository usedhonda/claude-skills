---
description: Generate agents and skills from conversation history
argument-hint: [--agents|--skills]
---

# learn-gen

Analyze conversation history, git commits, and codebase to generate project-specific agents and skills.

## Usage

```
/skill-from-history:learn-gen                # Generate both
/skill-from-history:learn-gen --agents       # Agents only
/skill-from-history:learn-gen --skills       # Skills only
```

## Process

```
1. Source Collection
   - Conversation history
   - Git commits
   - Codebase patterns

2. Agent Pattern Extraction
   - Role/persona detection

3. Skill Pattern Extraction
   - Workflow/procedure detection

4. Conflict Detection
   - Inter-agent conflicts
   - Inter-skill conflicts

5. Proposal Display
   - Candidates with confidence scores

6. User Selection -> Generation
```

## Output

```
.claude/
  agents/
    {name}/AGENT.md
  skills/
    {name}/SKILL.md
```

## See Also

- [learn-check](./learn-check.md) - Evaluate generated skills
- [learn-promote](./learn-promote.md) - Promote maturity level
