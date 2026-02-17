# Contributing to Agent Skills

Thanks for contributing! This guide covers the skill format and PR requirements for this repo, following the [Agent Skills Specification](https://agentskills.io/specification).

## Skill Format

Every skill is a directory under `skills/` with this structure:

```
skills/<name>/
  SKILL.md          # Required — skill definition
  README.md         # Required — user-facing docs
  tools/            # Optional — CLI tools
    <tool-name>     # Executable (bash, python, etc.)
```

### SKILL.md

The skill definition uses YAML frontmatter followed by markdown:

```yaml
---
name: my-skill
description: Use when user says "do the thing" or wants to accomplish X.
version: 1.0.0
---

# my-skill

One-line description of what this skill does.

## Overview

What the skill does and why it exists.

## When to Use

- Trigger phrases and scenarios

## Common Mistakes

**Mistake description.** How to avoid it.

## CLI Tool (if applicable)

### Quick Reference
### Commands
### Examples

## Data Locations (if applicable)

Where the skill reads/writes data.
```

### Required Sections

| Section | Purpose |
|---------|---------|
| Frontmatter | Name, description (trigger phrases), version |
| Overview | What and why |
| When to Use | Bullet list of trigger scenarios |
| Common Mistakes | Gotchas with bold headers |

### Optional Sections

- CLI Tool (with Quick Reference, Commands, Examples)
- Data Locations
- Integration notes

## CLI Tools

If your skill includes CLI tools:

- Place executables in `tools/` directory
- Include a shebang line (`#!/bin/bash`, `#!/usr/bin/env python3`)
- Tools must be self-contained (no external package managers)
- Include a `help` subcommand
- Use `set -e` in bash scripts

## Security

This repo has automated security checks at two levels:

**Pre-commit hooks** (local) — enable with `git config core.hooksPath .githooks`:
- Blocks commits containing API keys, tokens, private keys, and passwords
- Blocks hardcoded personal paths (`/Users/...`)
- Blocks dangerous file types (`.pem`, `.key`, `.env`, `credentials.*`)

**GitHub Actions** (CI) — runs on every push and PR:
- [gitleaks](https://github.com/gitleaks/gitleaks) secret scanning
- Custom internal reference detection
- Dangerous file detection

Both must pass before code can be merged.

## PR Checklist

Before submitting:

- [ ] Enabled pre-commit hooks: `git config core.hooksPath .githooks`
- [ ] No hardcoded user paths (`/Users/...`, `~/specific-project/`)
- [ ] No references to private systems or internal tooling
- [ ] No API keys, tokens, or credentials in any files
- [ ] SKILL.md has all required sections
- [ ] README.md with installation and usage instructions
- [ ] Tools are executable (`chmod +x`)
- [ ] `help` command works
- [ ] Tested on a clean install (copy to fresh `.claude/skills/`)

## Publishing from a Private Project

If you maintain a private Claude Code project with skills you want to contribute, [publish-skill](../skills/publish-skill/) automates the full workflow: stripping internal references, running security scans, and pushing to this repo.

It handles the common footguns automatically — hardcoded paths, project-specific names, runtime files — via a configurable sanitization pipeline with a mandatory security scan gate.

## Template

Use this as a starting point:

```bash
mkdir -p skills/my-skill/tools
cat > skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: Use when user says "my-skill" or wants to do X.
version: 1.0.0
---

# my-skill

Brief description.

## Overview

What this skill does.

## When to Use

- "Do the thing"

## Common Mistakes

**Common issue.** How to avoid it.
EOF
```
