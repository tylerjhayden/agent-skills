---
name: mde
description: Use when user says "open in markdown", "mde", "recent markdown files", "find my last markdown", or wants to open/find .md files.
version: 1.0.1
---

# mde

MacDown 3000 CLI with smart recent-file discovery.

## Overview

Opens markdown files in MacDown 3000 with commands for finding recently modified files across the home directory. Supports configurable directory exclusions and path normalization for absolute paths.

## When to Use

- "Open this file in MacDown 3000"
- "What markdown files did I edit recently?"
- "Open my most recent markdown file"
- "Exclude this directory from mde searches"

## Common Mistakes

**Using full paths in exclusions.** The tool normalizes them, but prefer short patterns like `archive` over `~/Documents/archive`.

## CLI Usage

`mde` is aliased in `~/.zshrc` to `<path-to-skill>/tools/mde`.

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

Hidden directories (`*/.*`) are always excluded automatically.

## Data Locations

| Operation | Path | Description |
|-----------|------|-------------|
| Reads/Writes | `~/.config/mde/config.json` (or `$MDE_CONFIG_DIR`) | MacDown 3000 CLI configuration |
