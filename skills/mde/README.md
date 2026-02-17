# mde

> MacDown 3000 CLI with smart recent-file discovery.
>
> Part of [agent-skills](https://github.com/tylerjhayden/agent-skills)

## Installation

Copy the skill directory into your Claude Code project:

```bash
# From the agent-skills repo
cp -r skills/mde .claude/skills/mde

# Or user-level (available across all projects)
cp -r skills/mde ~/.claude/skills/mde
```

## Requirements

- macOS
- [MacDown 3000](https://github.com/schuyler/macdown3000) — download and install the latest release before using this skill
- jq

## Setup

Alias the tool in your shell profile:

```bash
# Add to ~/.zshrc or ~/.bash_profile
alias mde="/path/to/.claude/skills/mde/tools/mde"
source ~/.zshrc
```

By default, config is stored at `~/.config/mde/config.json`. To store it elsewhere, set `MDE_CONFIG_DIR` before the alias:

```bash
export MDE_CONFIG_DIR="$HOME/.config/mde"   # default — change to taste
alias mde="/path/to/.claude/skills/mde/tools/mde"
```

The AI can invoke `mde` via its Bash tool without the alias. The alias is for you — so you can open files from your terminal the same way the AI does.

## Overview

Opens markdown files in MacDown 3000 with commands for finding recently modified files across the home directory. Supports configurable directory exclusions and path normalization for absolute paths.

## Quick Reference

```bash
mde                     # Open MacDown 3000
mde <file>              # Open specific file
mde last                # Open most recently modified .md
mde recent [N]          # List N most recent .md files (default: 5)
mde excludes            # Show exclusion patterns
mde exclude <pattern>   # Add exclusion
mde unexclude <pattern> # Remove exclusion
mde help                # Show help
```

## When to Use

- "Open this file in MacDown 3000"
- "What markdown files did I edit recently?"
- "Open my most recent markdown file"
- "Exclude this directory from mde searches"

## How Search Works

`mde` uses an **exclusion-first** approach: it searches your entire home directory by default, then prunes directories you specify. You don't have to enumerate every project folder — just tell it what to ignore.

Two knobs in `config.json`:

- **`searchRoot`** — where to start. Default `~`. Narrow to `~/Documents` or `~/Projects` if your markdown is well-organized and you want faster results.
- **`excludes`** — directory name patterns to skip. Matched anywhere in the path, so `node_modules` prunes every `node_modules/` across your whole tree.

Hidden directories (`*/.*`) are always excluded automatically.

Good starting exclusions:

```json
"excludes": ["Library", ".Trash", ".cache", "node_modules", ".git", "vendor", "archive"]
```

Prefer short, type-based patterns (`archive`, `vendor`, `build`) over project-specific paths. They stay accurate as your directory structure evolves.

## Configuration

Config file: `~/.config/mde/config.json` (override with `$MDE_CONFIG_DIR`)

```json
{
  "excludes": ["Library", ".Trash", "node_modules"],
  "searchRoot": "~",
  "maxDepth": 10,
  "defaultRecentCount": 5
}
```

You can also manage exclusions interactively via `mde exclude <pattern>` and `mde unexclude <pattern>` — no need to edit the JSON file directly.

## License

MIT — see [LICENSE](../../LICENSE).
