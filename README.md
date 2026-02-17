# Agent Skills

Curated Claude Code skills — ready to drop into any project.

Each skill is a self-contained directory with a `SKILL.md` definition and optional CLI tools. Copy a skill into your `.claude/skills/` directory and it works immediately.

## Skills

| Skill | Description | Platform | Version |
|-------|-------------|----------|---------|
| [bear](skills/bear/) | Bear.app CLI bridge — two-way sync between filesystem markdown and Bear notes. | macOS | 1.0.0 |
| [claude-usage](skills/claude-usage/) | Fetches real-time claude.ai session and weekly usage limits using headless Playwright to bypass Cloudflare TLS fingerprinting, then displays progress bars per model tier. Includes a background poller that keeps a cache file fresh for the statusline. | macOS | 1.0.2 |
| [mde](skills/mde/) | MacDown 3000 CLI with smart recent-file discovery. | macOS | 1.0.1 |

## Installation

1. Pick a skill from the catalog above
2. Copy its directory into your project or user skills:

```bash
# Project-level (recommended)
cp -r skills/bear .claude/skills/bear

# Or user-level (available across all projects)
cp -r skills/bear ~/.claude/skills/bear
```

3. The skill is now available in Claude Code sessions

## Skill Format

Every skill follows the [Agent Skills Specification](CONTRIBUTING.md):

```
skills/<name>/
  SKILL.md          # Skill definition (YAML frontmatter + markdown)
  README.md         # User-facing documentation
  tools/            # Optional CLI tools (bash, python, etc.)
```

## Development Setup

If you're contributing, enable the pre-commit security hooks:

```bash
git config core.hooksPath .githooks
```

This runs automatic checks for secrets, hardcoded paths, and internal references before every commit.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the skill format spec and PR checklist.

## License

MIT — see [LICENSE](LICENSE).

---
