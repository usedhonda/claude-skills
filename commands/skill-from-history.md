---
description: Generate project-specific skills from Claude Code history and git commits
argument-hint: [options]
---

# Skill from History Generator

Analyze Claude Code conversation history, git commits, and codebase patterns to generate reusable project-specific skills.

This command invokes the skill-from-history skill to:
1. Scan conversation history at `~/.claude/projects/`
2. Analyze git commits and codebase patterns
3. Detect recurring patterns and anti-patterns
4. Generate new skills in `.claude/skills/`

Usage:
- `/skill-from-history` - Full analysis and generation
