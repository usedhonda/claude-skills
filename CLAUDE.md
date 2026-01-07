# claude-skills Development Guide

Claude Code plugin marketplace for project-specific skills.

## Command Structure (v4.2.0)

### Plugin-Specific Commands

Each plugin has its own command namespace to avoid duplication:

```
skills/skill-from-history/commands/
├── learn-status.md
├── learn-init.md
├── learn-gen.md
├── learn-check.md
└── learn-promote.md

skills/parallel-dev-orchestrator/commands/
├── par-status.md
├── par-init.md
├── par-plan.md
├── par-dispatch.md
└── par-harvest.md
```

### skill-from-history Commands

```
/skill-from-history:learn-status   # Show current status
/skill-from-history:learn-init     # Initialize Golden Tasks
/skill-from-history:learn-gen      # Generate agents/skills
/skill-from-history:learn-check    # Run evaluation
/skill-from-history:learn-promote  # Promote maturity level
```

### parallel-dev-orchestrator Commands

```
/parallel-dev-orchestrator:par-status   # Show PR status
/parallel-dev-orchestrator:par-init     # Check environment
/parallel-dev-orchestrator:par-plan     # Create task plan
/parallel-dev-orchestrator:par-dispatch # Dispatch tasks
/parallel-dev-orchestrator:par-harvest  # Harvest and merge PRs
```

### Quick Start

```bash
# Knowledge Management
/skill-from-history:learn-status    # Check current state
/skill-from-history:learn-init      # Setup Golden Tasks
/skill-from-history:learn-gen       # Generate skills

# Parallel Development
/parallel-dev-orchestrator:par-init      # Check prerequisites
/parallel-dev-orchestrator:par-plan      # Create task plan
/parallel-dev-orchestrator:par-dispatch  # Submit tasks
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
  ]
}
```

**Note**: Each skill has its own `commands/` directory. Commands use prefixes (`learn-*`, `par-*`) to indicate which skill they belong to.

### 5. Update marketplace.json

**CRITICAL**: Each plugin MUST have `commands` and `agents` fields, or commands won't work in other projects!

```json
{
  "plugins": [
    {
      "name": "{skill-name}",
      "description": "{one-line description}",
      "source": "./",
      "skills": ["./skills/{skill-name}"],
      "commands": ["./skills/{skill-name}/commands/"],
      "agents": ["./agents/{agent-name}/AGENT.md"]
    }
  ]
}
```

| Field | Required | Notes |
|-------|----------|-------|
| `name` | ✅ | Must match skill directory name |
| `description` | ✅ | One-line description |
| `source` | ✅ | Always `"./"` |
| `skills` | ✅ | Path to SKILL.md directory |
| `commands` | ✅ | Path to commands/ directory (empty array `[]` if none) |
| `agents` | ✅ | **Must end with `.md`** (e.g., `./agents/name/AGENT.md`) |

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

**Run this BEFORE every commit:**

```bash
# === CRITICAL: Config Consistency Check ===
echo "=== Version Check ==="
settings_ver=$(jq -r '.version' .claude-plugin/settings.json)
marketplace_ver=$(jq -r '.metadata.version' .claude-plugin/marketplace.json)
if [ "$settings_ver" != "$marketplace_ver" ]; then
  echo "❌ VERSION MISMATCH: settings=$settings_ver marketplace=$marketplace_ver"
  exit 1
else
  echo "✅ Version: $settings_ver"
fi

echo ""
echo "=== marketplace.json Plugin Fields Check ==="
jq -r '.plugins[] | "\(.name): commands=\(.commands // "MISSING!") agents=\(.agents // "MISSING!")"' .claude-plugin/marketplace.json

echo ""
echo "=== Commands Path Validation ==="
for plugin in $(jq -r '.plugins[].name' .claude-plugin/marketplace.json); do
  cmd_path=$(jq -r --arg p "$plugin" '.plugins[] | select(.name==$p) | .commands[0]' .claude-plugin/marketplace.json)
  if [ "$cmd_path" = "null" ] || [ -z "$cmd_path" ]; then
    echo "❌ $plugin: commands field MISSING in marketplace.json!"
  elif [ ! -d "${cmd_path#./}" ]; then
    echo "❌ $plugin: commands path not found: $cmd_path"
  else
    count=$(ls "${cmd_path#./}"/*.md 2>/dev/null | wc -l)
    echo "✅ $plugin: $count commands in $cmd_path"
  fi
done

echo ""
echo "=== Command Duplication Check ==="
for cmd in $(ls skills/*/commands/*.md 2>/dev/null | xargs -n1 basename | sort | uniq -d); do
  echo "❌ DUPLICATE: $cmd"
  find . -name "$cmd" -type f
done

echo ""
echo "=== Skills ↔ Agents Cross-Reference ==="
grep -h "agents:" skills/*/SKILL.md 2>/dev/null | grep -v "^#"
grep -h "skills:" agents/*/AGENT.md 2>/dev/null | grep -v "^#"
```

### Quick Validation (One-liner)

```bash
# Fast check - run before git push
missing=$(jq '[.plugins[] | select(.commands == null or .agents == null) | .name]' .claude-plugin/marketplace.json)
[ "$missing" = "[]" ] && echo "✅ OK" || echo "❌ FAIL: missing commands/agents: $missing"
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

# List skill-specific commands
ls -la skills/*/commands/

# Test commands
/skill-from-history:learn-status
/parallel-dev-orchestrator:par-status
```

---

## Quick Reference

### Add a New Skill

```bash
# 1. Create structure
mkdir -p skills/{name}/references

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

# 3. Create commands/ directory with {prefix}-*.md files
# 4. Update settings.json (add to skills[], commands[])
# 5. Update marketplace.json (add to plugins[])
# 6. Bump version in both files
```

### Add a Command to Existing Skill

```bash
# Create new command file in skill's commands/ directory
# skills/{skill-name}/commands/{prefix}-{action}.md

# Example: Add learn-export to skill-from-history
cat > skills/skill-from-history/commands/learn-export.md << 'EOF'
---
description: Export skills to external format
argument-hint: [--format json|yaml]
---

# learn-export

Documentation...
EOF
```

---

## Evidence Index

| Issue | Commit | Resolution |
|-------|--------|------------|
| Commands duplicated across plugins | 2b7f647 | Remove from root `commands/` |
| Commands in wrong namespace | f240516 | Restore to skill-specific paths |
| Plugin source overlap | - | All use `source: "./"` - commands must be in ONE location |
| Command fragmentation (v4.0.0) | - | Unified into `/cs-learn-skills` and `/cs-run-parallel` |
| Command duplication (v4.1.0) | - | Split to plugin-specific: `learn-*` / `par-*` |
| **marketplace.json missing commands/agents** | 33d2fef | Add `commands` and `agents` fields to each plugin (v4.2.0) |
| **agents path must end with .md** | - | Use `./agents/name/AGENT.md` not `./agents/name` (v4.2.1) |
