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

## Overview


Opens markdown files in MacDown 3000 with commands for finding recently modified files across the home directory. Supports configurable directory exclusions and path normalization for absolute paths.
## When to Use


- "Open this file in MacDown 3000"
- "What markdown files did I edit recently?"
- "Open my most recent markdown file"
- "Exclude this directory from mde searches"
## License

MIT — see [LICENSE](../../LICENSE).
