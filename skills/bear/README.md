# bear

> Bear.app CLI bridge — two-way sync between filesystem markdown and Bear notes.
>
> Part of [agent-skills](https://github.com/tylerjhayden/agent-skills)

## Installation

Copy the skill directory into your Claude Code project:

```bash
# From the agent-skills repo
cp -r skills/bear .claude/skills/bear

# Or user-level (available across all projects)
cp -r skills/bear ~/.claude/skills/bear
```

## Requirements

- macOS
- Bear.app
- sqlite3
- python3

## Overview


Sends markdown files into Bear for editing (via URL scheme) and pulls Bear notes back to the filesystem as clean markdown (via read-only SQLite queries). Import never touches the database directly — it uses Bear's URL scheme to preserve CloudKit sync. Export is always read-only.
## When to Use


- "Send this to Bear" / "Open in Bear"
- "Pull my Bear notes" / "Export from Bear"
- "What are my recent Bear notes?"
- "Find Bear notes tagged with X"
- "Open this note in Bear"
## Quick Reference


```bash
bear <file.md>                # Send file to Bear (shortcut for send)
bear send <file.md>           # Send/update markdown file in Bear
bear pull <title> [path]      # Pull Bear note to filesystem
bear pull-all [path]          # Export all notes (default: ./bear-export/)
bear open <title>             # Open note in Bear
bear search <query>           # Search in Bear
bear recent [N]               # List N most recent notes (default: 10)
bear list [--tag <tag>]       # List notes, optionally by tag
bear help                     # Show help
```

### Commands

| Command | Description |
|---------|-------------|
| `send <file>` | Send markdown file to Bear. Updates existing note if title matches, creates new note with project tag if not |
| `pull <title> [path]` | Export note as clean markdown. Resolves `[image:UUID/file]` to `![](./images/file)` and copies image files |
| `pull-all [path]` | Bulk export all non-trashed notes to directory |
| `open <title>` | Open note directly in Bear by title |
| `search <query>` | Open Bear's search UI with query |
| `recent [N]` | List N most recently modified notes with relative timestamps |
| `list [--tag tag]` | List all notes or filter by tag |
## Examples


**Edit a project doc in Bear's editor, then pull it back:**
```bash
bear send docs/architecture.md     # Push to Bear (tagged #my-project)
# ... edit in Bear's markdown editor ...
bear pull "Architecture" docs/architecture.md   # Pull back with clean markdown
```

**Quick lookup of recent notes:**
```bash
bear recent 5                      # What have I been working on?
bear list --tag work            # All project-related notes
bear open "Meeting Notes"          # Jump straight into Bear
```

**Bulk export for backup or migration:**
```bash
bear pull-all ~/Desktop/bear-backup/   # Export everything with images
```

### Integration

- Project tags auto-derived from your directory structure (`~/my-project/` -> `#my-project`, `~/Projects/unbound/` -> `#unbound`)
- Stateless — no state files, so no conflicts with other skills
## License

MIT — see [LICENSE](../../LICENSE).
