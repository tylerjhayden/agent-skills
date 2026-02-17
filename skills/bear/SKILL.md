---
name: bear
description: Use when user says "bear", "send to bear", "pull from bear", "bear notes", "export bear", or wants to bridge Bear.app notes with the filesystem.
version: 1.0.0
---

# bear

Bear.app CLI bridge — two-way sync between filesystem markdown and Bear notes.

## Overview

Sends markdown files into Bear for editing (via URL scheme) and pulls Bear notes back to the filesystem as clean markdown (via read-only SQLite queries). Import never touches the database directly — it uses Bear's URL scheme to preserve CloudKit sync. Export is always read-only.

## When to Use

- "Send this to Bear" / "Open in Bear"
- "Pull my Bear notes" / "Export from Bear"
- "What are my recent Bear notes?"
- "Find Bear notes tagged with X"
- "Open this note in Bear"

## Common Mistakes

**Quoting titles with special characters.** Always wrap Bear note titles in quotes when they contain spaces or punctuation: `bear open "My Note Title"`.

**Expecting instant sync on send.** Bear processes URL scheme requests asynchronously. The note may take a moment to appear after `bear send`.

**Running pull-all into a tracked directory.** Export dumps many files — use a scratch directory or `./bear-export/` (the default) to avoid polluting git repos.

## CLI Tool

**Alias:** `bear`
**Location:** `.claude/skills/bear/tools/bear`

### Quick Reference

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

### Send Behavior

- Title extracted from first `# H1` heading, falls back to filename
- If note with same title exists: updates via `replace_all` mode
- If new: creates with project tag derived from file path (`/my-project/...` -> `#my-project`)

### Pull Behavior

- Bear's `[image:UUID/filename.png]` syntax converted to standard `![](./images/filename.png)`
- Image files copied from Bear's local storage to `./images/` alongside the output file
- YAML frontmatter preserved if present in the note (never added)

### Output Format

**Default:** Plain text (numbered lists, formatted tables)
- `recent` and `list` output numbered lists to stdout
- `send`, `pull`, `open` output status messages

### Examples

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

## Data Locations

| Operation | Path | Description |
|-----------|------|-------------|
| Reads | `~/Library/Group Containers/9K33E3U3T4.net.shinyfrog.bear/Application Data/database.sqlite` | Bear SQLite database (read-only) |
| Reads | `~/Library/Group Containers/9K33E3U3T4.net.shinyfrog.bear/Application Data/Local Files/Note Images/` | Bear image assets |
| N/A | Stateless tool | Stateless tool |
