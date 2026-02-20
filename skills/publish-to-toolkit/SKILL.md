---
name: publish-to-toolkit
description: Publish skills and tools to the public agent-toolkit repo. Use when user says "publish skill", "push to marketplace", or wants to share skills publicly.
metadata:
  version: 1.0.0
  portability:
    safe_for_linking: true
    required_env:
      - PROJECT_HOME
    writes_to:
      - ~/your-agent-toolkit-repo/ (public repo)
    reads_from:
      - $PROJECT_HOME/.claude/skills/<name>/
    notes: "Publishes skills to ~/your-agent-toolkit-repo public repo. Uses PROJECT_HOME for source. Requires git and agent-toolkit repo. Fully portable."
---

# publish-to-toolkit

Sanitize and publish your skills and tools to the public agent-toolkit GitHub repo.

## Overview

Automates the process of taking a private skill or tool, stripping internal references (paths, names, cross-references), generating user-facing docs, and pushing to the `your-username/agent-toolkit` public repo. Tracks versions and file hashes to detect changes.

## When to Use

- "Publish bear to the marketplace"
- "Push this skill public"
- "What skills are published?"
- "Diff bear against the published version"

## README vs SKILL.md — Two-Audience Doctrine

These files serve different readers and should be authored differently:

| | SKILL.md | README.md |
|---|---|---|
| **Audience** | The AI agent | Humans installing the skill |
| **Tone** | Dense, operational | Explanatory, setup-focused |
| **Must include** | When to invoke, commands, data locations | Installation, shell alias setup, why it works the way it does |
| **Should include** | Common mistakes (for the agent) | First-time config steps, common mistakes, full command ref |
| **Should NOT include** | Verbose rationale, setup prose | Internal implementation details |

**SKILL.md is optimized for context.** Every line burns tokens. Keep it tight: trigger phrases, command reference, data locations.

**README.md must be useful before the AI is involved.** A human follows it to install the skill — they can't ask the AI for help yet. It must answer: *What is this? Why does it work this way? How do I get it running?*

Skills with CLI tools nearly always need three things in the README that `generate_readme` cannot produce:
1. **Shell alias setup** — how the tool gets on PATH for terminal use
2. **"Why" explanations** — rationale for non-obvious design choices (e.g., "Why Playwright?", "Why URL scheme?")
3. **First-time credential/config steps** — concrete instructions for acquiring API keys, session cookies, etc.

When a README needs this level of detail, set `preserve_readme: true` in the manifest. The publish flow will preserve the handcrafted README instead of regenerating it from SKILL.md.

## Common Mistakes

**Publishing without `init` first.** Skills must be initialized in the manifest before publishing. Run `publish-to-toolkit init <name>` to add a skill to the manifest with default config.

**Forgetting per-skill strip rules.** Default sanitization handles common patterns, but each skill may have unique internal references. Use `--dry-run` to inspect sanitized output before committing.

**Handcrafted README getting overwritten.** If a skill has a detailed, human-authored README, set `preserve_readme: true` in the manifest before publishing. Without it, `generate_readme` overwrites the README with a sparse auto-generated version from SKILL.md.

## CLI Tool

**Alias:** `publish-to-toolkit`
**Requirements:** bash, jq, rsync, git

### Quick Reference

```bash
publish-to-toolkit publish <name>             # Sanitize + scan + copy + commit + push
publish-to-toolkit publish <name> --dry-run   # Sanitize + scan only — no commit or push
publish-to-toolkit publish <name> --force     # Publish even if source is unchanged
publish-to-toolkit list                       # Show all skills and publish status
publish-to-toolkit diff <name>                # Diff local vs published version
publish-to-toolkit init <name>                # Mark a skill as publishable
publish-to-toolkit help                       # Show help
```

### Commands

| Command | Description |
|---------|-------------|
| `publish <name>` | Sanitize skill/tool, security scan, copy to agent-toolkit, bump version, commit and push |
| `publish <name> --dry-run` | Sanitize and scan only — inspect output without committing |
| `publish <name> --force` | Publish even if source hashes are unchanged |
| `list` | Show all skills in manifest with version and publish status |
| `diff <name>` | Show diff between local source and published version |
| `init <name>` | Add a skill to the manifest as publishable |
| `help` | Show usage |

### Publish Flow

1. Verify skill exists in manifest
2. Compute SHA256 hashes of all source files
3. Skip if unchanged (use `--force` to override)
4. Copy skill to `~/your-agent-toolkit-repo/<category>/<name>/`
5. Apply sanitization (default rules + per-skill overrides)
6. If `category: "tools"`: remove SKILL.md from published artifact
7. Run security scan — **hard stop** on any finding (all failures collected before aborting)
8. Generate README.md from sanitized SKILL.md (skipped for tools — must use `preserve_readme: true`)
9. Prompt for version bump (patch/minor/major)
10. Update manifest with hashes, version, timestamp
11. Update top-level README.md catalog table (Skills or Tools table based on category)
12. Git commit and push in agent-toolkit repo

> With `--dry-run`, the flow stops after step 6. No version bump, no commit, no push. Use this to inspect sanitized output before committing to a release.

## Data Locations

| Operation | Path | Description |
|-----------|------|-------------|
| Config | `.claude/skills/publish-to-toolkit/publish-manifest.json` | Publish config, versions, hashes |
| Target | `target_repo` in manifest | Public repo (push target) — `~/your-agent-toolkit-repo` |
| Source | `.claude/skills/<name>/` | Skill source directory (read-only) |
| Category | `category` field per skill in manifest | `"skills"` (default) or `"tools"` — determines target subdirectory and whether SKILL.md is stripped |
