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

## How search works

`mde` uses an **exclusion-first** approach: it searches your entire home directory by default, then prunes directories you specify. You don't have to enumerate every project folder — just tell it what to ignore.

Two knobs in `config.json`:

- **`searchRoot`** — where to start. Default `~`. Narrow to `~/Documents` or `~/Projects` if your markdown is well-organized and you want faster results.
- **`excludes`** — directory name patterns to skip. Matched anywhere in the path, so `node_modules` prunes every `node_modules/` across your whole tree.

Good starting exclusions:

```json
"excludes": ["Library", ".Trash", ".cache", "node_modules", ".git", "vendor", "archive"]
```

Prefer short, type-based patterns (`archive`, `vendor`, `build`) over project-specific paths. They stay accurate as your directory structure evolves.

## Overview

Opens markdown files in MacDown 3000 with commands for finding recently modified files across the home directory. Supports configurable directory exclusions and path normalization for absolute paths.

## When to Use

- "Open this file in MacDown 3000"
- "What markdown files did I edit recently?"
- "Open my most recent markdown file"
- "Exclude this directory from mde searches"

## License

MIT — see [LICENSE](../../LICENSE).
