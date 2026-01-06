# claude-skills Development Guide

Claude Code plugin marketplace for project-specific skills.

## Critical: Command Placement Rules

**Root Problem**: Multiple plugins with `source: "./"` load ALL commands, causing duplication.

### Rule 1: One Location Only

| Command Type | Location | Example |
|--------------|----------|---------|
| **Shared** (used by ALL skills) | `commands/` | gen-all, gen-agents, gen-skills |
| **Skill-specific** | `skills/{name}/commands/` | skill-eval, orchestrate, dispatch |

**NEVER place the same command in both locations.**

### Rule 2: Current Command Map

```
commands/                              # Shared (meta-generation)
├── gen-all.md
├── gen-agents.md
└── gen-skills.md

skills/skill-from-history/commands/    # skill-from-history only
├── skill-eval.md
└── skill-promote.md

skills/parallel-dev-orchestrator/commands/  # parallel-dev-orchestrator only
├── orchestrate.md
├── dispatch.md
├── harvest.md
└── plan-parallel.md
```

---

## New Skill Checklist

### 1. Create Directory Structure

```
skills/{skill-name}/
├── SKILL.md           # REQUIRED
├── commands/          # REQUIRED if skill has commands
│   └── {command}.md
├── README.md          # Recommended
├── references/        # Recommended
└── templates/         # Optional
```

### 2. SKILL.md Format

```yaml
---
name: {skill-name}
description: |
  One-line description.
  Trigger phrases here.
compression-anchors:
  - "Key concept 1"
  - "Key concept 2"
  - "Key concept 3"
agents:                # Optional
  - {agent-name}
---

# Skill Name

Content...
```

### 3. Command File Format

```yaml
---
description: One-line description for /help
argument-hint: [optional args]
---

# Command Name

Usage and documentation...
```

### 4. Update settings.json

```json
{
  "skills": [
    { "name": "{skill-name}", "path": "skills/{skill-name}" }
  ],
  "commands": [
    "./skills/{skill-name}/commands/"
  ]
}
```

### 5. Update marketplace.json

```json
{
  "plugins": [
    {
      "name": "{skill-name}",
      "description": "{one-line description}",
      "source": "./",
      "skills": ["./skills/{skill-name}"]
    }
  ]
}
```

### 6. Bump Version

Both files must have matching version:
- `.claude-plugin/settings.json`
- `.claude-plugin/marketplace.json`

---

## Version Management

| Change Type | Version Part | Example |
|-------------|--------------|---------|
| New skill added | MINOR (x.Y.z) | 3.4.0 → 3.5.0 |
| New command in existing skill | PATCH (x.y.Z) | 3.4.0 → 3.4.1 |
| Bug fix / docs | PATCH (x.y.Z) | 3.4.0 → 3.4.1 |
| Breaking change | MAJOR (X.y.z) | 3.4.0 → 4.0.0 |

---

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Skill name | lowercase-with-hyphens (max 32 chars) | `parallel-dev-orchestrator` |
| Command name | verb-first-action | `dispatch`, `harvest` |
| Agent name | lowercase-with-hyphens | `worktree-dispatcher` |
| Branch name | `feat/{skill-name}-{feature}` | `feat/skill-from-history-lint` |

---

## Pre-Release Checklist

```bash
# 1. Check for command duplication
for cmd in $(ls skills/*/commands/*.md | xargs -n1 basename | sort -u); do
  count=$(find . -name "$cmd" -type f | wc -l)
  if [ "$count" -gt 1 ]; then
    echo "DUPLICATE: $cmd"
    find . -name "$cmd" -type f
  fi
done

# 2. Verify SKILL.md frontmatter
for skill in skills/*/SKILL.md; do
  echo "=== $skill ==="
  head -10 "$skill"
done

# 3. Cross-reference check
echo "=== Agents referenced in skills ==="
grep -r "agents:" skills/*/SKILL.md

echo "=== Skills referenced in agents ==="
grep -r "skills:" agents/*/AGENT.md
```

---

## Troubleshooting

### Commands Not Appearing in Suggestions

| Symptom | Cause | Solution |
|---------|-------|----------|
| Not appearing after install | Plugin cache | Restart Claude Code completely |
| Still old after update | Marketplace cache | `/plugin` → Update marketplace |
| Specific command missing | YAML frontmatter error | Check `---` closing tag |

### Marketplace Update Not Reflecting

```bash
# Resolution steps
1. /plugin → Marketplaces → Update marketplace
2. Restart Claude Code
3. If still broken: /plugin uninstall → /plugin install
```

### GitHub CDN Cache (~15 min)

- `raw.githubusercontent.com` has CDN caching
- Changes may not reflect immediately after push
- Wait 15 minutes or do full reinstall

### Debug Commands

```bash
# Check plugin status
/plugin

# Verify settings.json
cat .claude-plugin/settings.json | jq '.commands'

# Check for duplicates
for cmd in plan-parallel orchestrate dispatch harvest; do
  echo "=== $cmd ==="
  find . -name "${cmd}.md" -type f
done
```

---

## Quick Reference

### Add a New Skill

```bash
# 1. Create structure
mkdir -p skills/{name}/commands skills/{name}/references

# 2. Create SKILL.md
cat > skills/{name}/SKILL.md << 'EOF'
---
name: {name}
description: Description here
compression-anchors:
  - "Key concept"
---

# Skill Name

Content...
EOF

# 3. Update settings.json (add to skills[] and commands[])
# 4. Update marketplace.json (add to plugins[])
# 5. Bump version in both files
# 6. Run pre-release checklist
```

### Add a Command to Existing Skill

```bash
# 1. Create command file
cat > skills/{skill-name}/commands/{command}.md << 'EOF'
---
description: Brief description
argument-hint: [optional args]
---

# Command Name

Documentation...
EOF

# 2. Verify no duplication in root commands/
ls commands/{command}.md 2>/dev/null && echo "ERROR: Duplicate exists!"

# 3. Bump patch version
```

---

## Evidence Index

| Issue | Commit | Resolution |
|-------|--------|------------|
| Commands duplicated across plugins | 2b7f647 | Remove from root `commands/` |
| Commands in wrong namespace | f240516 | Restore to skill-specific paths |
| Plugin source overlap | - | All use `source: "./"` - commands must be in ONE location |
