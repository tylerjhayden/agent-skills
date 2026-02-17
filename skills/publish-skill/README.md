# publish-skill

> Sanitize and publish your private Claude Code skills to a public `agent-skills` repo.
>
> Part of [agent-skills](https://github.com/your-username/agent-skills)

## What it does

`publish-skill` automates the full contribution pipeline from private project to public repo:

1. **Copy** the skill directory to your public `agent-skills` repo
2. **Sanitize** — strip internal paths, project names, and private references via configurable replacement rules
3. **Exclude** — omit private config files entirely (e.g. your own `publish-manifest.json`)
4. **Security scan** — hard stop on any PII, API keys, hardcoded paths, or runtime files that slipped through
5. **Generate or preserve README** — auto-generate from `SKILL.md`, or preserve a handcrafted one
6. **Version bump** — patch/minor/major, tracked in the manifest
7. **Commit and push** — targeted `git add` on the skill directory, never `git add -A`

The `--dry-run` flag stops after the security scan so you can inspect sanitized output before committing anything.

## Requirements

- bash
- jq
- git
- A public GitHub repo cloned locally (your `agent-skills` fork or equivalent)

## Installation

Copy the skill into your Claude Code project:

```bash
# Project-level
cp -r skills/publish-skill .claude/skills/publish-skill

# Or user-level (available across all projects)
cp -r skills/publish-skill ~/.claude/skills/publish-skill
```

Add the CLI tool to your PATH. Add one of the following to your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
# If installed at project level
alias publish-skill="/path/to/your-project/.claude/skills/publish-skill/tools/publish-skill"

# If installed at user level
alias publish-skill="$HOME/.claude/skills/publish-skill/tools/publish-skill"
```

Then reload your shell:

```bash
source ~/.zshrc
```

## Setup

### 1. Create your public repo

Create a GitHub repo (e.g. `your-username/agent-skills`) and clone it locally:

```bash
git clone https://github.com/your-username/agent-skills ~/your-agent-skills-repo
```

### 2. Configure the manifest

Copy the example manifest and customize it:

```bash
cp .claude/skills/publish-skill/publish-manifest.example.json \
   .claude/skills/publish-skill/publish-manifest.json
```

Edit `publish-manifest.json`:
- Set `target_repo` to the path of your cloned public repo
- Update `default_replacements` with your project's internal names and paths
- Leave `skills: {}` empty — `init` populates it

### 3. Initialize your first skill

```bash
publish-skill init my-skill
```

This adds `my-skill` to the manifest with default config. Open `publish-manifest.json` and add any `extra_replacements`, `strip_lines`, or `exclude_files` needed for that skill.

### 4. Dry run

```bash
publish-skill publish my-skill --dry-run
```

Inspect the sanitized output at `~/your-agent-skills-repo/skills/my-skill/`. Confirm no private references remain.

### 5. Publish

```bash
publish-skill publish my-skill
```

Bumps the patch version, commits, and pushes to your public repo.

## Command Reference

| Command | Description |
|---------|-------------|
| `publish-skill publish <name>` | Full publish: sanitize, scan, copy, version bump, commit, push |
| `publish-skill publish <name> --dry-run` | Sanitize and scan only — no commit or push |
| `publish-skill publish <name> --force` | Publish even if source files are unchanged |
| `publish-skill publish <name> --bump minor` | Bump minor version instead of patch |
| `publish-skill list` | Show all skills with version and publish status |
| `publish-skill diff <name>` | Diff local source against published version |
| `publish-skill init <name>` | Add a skill to the manifest |
| `publish-skill help` | Show usage |

## Manifest Configuration

### Top-level fields

| Field | Description |
|-------|-------------|
| `target_repo` | Local path to your public agent-skills repo |
| `project_name` | Your private project's internal name; the scan blocks publish if any file still contains it |
| `skills_source` | Relative path to skills directory (default: `.claude/skills`) |
| `default_replacements` | `[{old, new}]` — applied to every published skill |

### Per-skill fields

| Field | Type | Description |
|-------|------|-------------|
| `published` | boolean | `true` once the skill has been pushed at least once |
| `version` | semver | Bumped automatically on each publish |
| `last_published` | ISO 8601 | Set automatically |
| `files_hash` | object | SHA256 map of source files, used for change detection |
| `requirements` | string[] | Shown in the auto-generated README Requirements section |
| `preserve_readme` | boolean | If `true`, your handcrafted `README.md` is kept as-is |
| `exclude_files` | string[] | Relative paths to omit entirely from the published output |
| `strip_lines` | string[] | Lines containing any of these strings are deleted from all files |
| `extra_replacements` | `[{old, new}]` | Applied after `default_replacements` |

## Security Scan

The security scan runs after sanitization and blocks publish on any of:

| Check | Blocks on |
|-------|-----------|
| Internal project name | Any file containing the value of `project_name` from your manifest (if set) |
| Personal paths | `/Users/<anything>` |
| Email addresses | Any email (except `noreply@anthropic.com`) |
| API keys / tokens | `ghp_*`, `ghs_*`, `AKIA*`, `sk-*`, `Bearer <long>` |
| Runtime files | `browser-state.json`, `cache.json`, `*.tmp` |

All failures are collected before aborting — you see everything that needs fixing in one pass.

If the scan blocks your publish, either:
- Add a replacement rule to `extra_replacements` to sanitize the value
- Add the file to `exclude_files` to omit it entirely
- Add the offending line pattern to `strip_lines` to delete it

## Two-Audience Doctrine: SKILL.md vs README.md

These files serve different readers:

| | SKILL.md | README.md |
|---|---|---|
| **Audience** | The AI agent | Humans installing the skill |
| **Tone** | Dense, operational | Explanatory, setup-focused |
| **Contains** | Trigger phrases, commands, data locations | Installation, PATH setup, first-time config |

**SKILL.md burns tokens.** Every line costs context window. Keep it tight.

**README.md must work before the AI is involved.** A human follows it to install the skill — they can't ask Claude for help yet.

For skills with CLI tools, set `preserve_readme: true` in the manifest. The auto-generated README from `SKILL.md` is always too sparse for tools that need shell aliases and first-time config steps.

## Contributing

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for the skill format and PR checklist.
