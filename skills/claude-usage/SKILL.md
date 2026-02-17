---
name: claude-usage
description: Use when user says "claude usage", "check usage", "how much context left",
  "session limit", "usage limits", "quota", "when does my session reset", or wants
  to know their current claude.ai usage percentage.
version: 1.0.2
---

# claude-usage

Fetches real-time claude.ai session and weekly usage limits using headless Playwright to bypass Cloudflare TLS fingerprinting, then displays progress bars per model tier. Includes a background poller that keeps a cache file fresh for the statusline.

## What This Skill Does

- **Session usage:** 5-hour rolling window utilization and reset time
- **Weekly usage:** 7-day aggregate across all models, Sonnet, and Opus tiers
- **Human display:** Progress bars with percentage and countdown to reset
- **JSON output:** Raw API response for scripting or inspection
- **Cache mode:** Writes structured JSON to `runtime/claude-usage/cache.json` for statusline
- **Background poller:** LaunchAgent fires every 2 min (session-aware + adaptive), updates cache automatically

## When to Use

**Trigger Phrases:**
- "claude usage"
- "check usage"
- "how much context left"
- "session limit" / "usage limits"
- "quota"
- "when does my session reset"
- "am I close to my limit"

**Context Clues:**
- User asks about rate limiting mid-session
- User wants to know if they can start a long task
- User suspects they're near the model limit

## CLI Commands

```bash
# First-time setup — prompts for sessionKey and orgId, stores in Keychain
claude-usage setup

# Check usage (human-readable progress bars)
claude-usage

# Raw JSON output (full API response)
claude-usage --json

# Write cache file (used by poller / statusline)
claude-usage --cache
```

**Output example:**
```
claude.ai usage — 10:42 AM

  Current session (5h)
  [████████████░░░░░░░░░░░░░░░░] 43%  resets in 2h 17m

  All models (7d)
  [████████░░░░░░░░░░░░░░░░░░░░] 28%  resets in 4d 6h

  Sonnet only (7d)
  [█████░░░░░░░░░░░░░░░░░░░░░░░] 19%  resets in 4d 6h
```


## Skill Limitations

1. **sessionKey expires in ~30 days.** When it does, `claude-usage setup` must be re-run to refresh Keychain credentials.

2. **Cloudflare dependency.** The first cold fetch hits `claude.ai` to acquire Cloudflare clearance. This takes 5–10 seconds. Subsequent fetches within 90 minutes use cached browser state and skip the warm-up (~2–4 seconds).

3. **Org-scoped only.** Fetches usage for the stored `org-id`. Multi-org setups require re-running setup.

## Data Locations

| Operation | Path | Description |
|-----------|------|-------------|
| Reads | macOS Keychain `claude-usage/session-key` | claude.ai `sessionKey` cookie (~30-day validity) |
| Reads | macOS Keychain `claude-usage/org-id` | claude.ai organization UUID |
| Writes | macOS Keychain `claude-usage/session-key` | Stored during `claude-usage setup` |
| Writes | macOS Keychain `claude-usage/org-id` | Stored during `claude-usage setup` |
| Writes | `runtime/claude-usage/browser-state.json` | Playwright storage state (Cloudflare clearance, 90-min TTL) |
| Writes | `runtime/claude-usage/cache.json` | Latest usage snapshot for statusline |
| Reads | `runtime/claude-usage/cache.json` | Read by your statusline script |
| Logs | `logs/claude-usage-poller.log` | Poller operational log (started, OK, ERROR) |
| Logs | `logs/claude-usage-poller-error.log` | stderr from claude-usage in poller mode |

Runtime files (`browser-state.json`, `cache.json`) are never committed to git.

## Background Poller

The LaunchAgent `com.myproject.claude-usage-poller` fires every 2 minutes. The poller script exits in <100ms when Claude Code isn't running. When active, it fetches based on utilization:

- **< 80% utilization** → fetch every ~10 min (cache freshness gate)
- **≥ 80% utilization** → fetch every ~2 min (high-urgency mode)
- **Session window expired** → immediate force-fetch regardless of cache age

```bash
# Load (one-time, or after plist changes)
launchctl load ~/Library/LaunchAgents/com.myproject.claude-usage-poller.plist

# Manually trigger a poll
launchctl start com.myproject.claude-usage-poller

# Watch the log
tail -f ~/my-project/logs/claude-usage-poller.log

# Unload (disable)
launchctl unload ~/Library/LaunchAgents/com.myproject.claude-usage-poller.plist
```

## Error Recovery — Session Expiry (~30-day cycle)

When `sessionKey` expires, the signal chain surfaces it at every level:

1. **Poller error log** (`logs/claude-usage-poller-error.log`):
   ```
   [SESSION EXPIRED] Run: claude-usage setup
   Steps: open https://claude.ai/settings/usage → DevTools → Application
          → Cookies → claude.ai → copy sessionKey value
   ```
2. **Cache file** contains `"error": "session_expired"`
3. **Status bar** shows `| ⚠ usage:expired`
4. **Fix**: `claude-usage setup` — re-stores fresh sessionKey in Keychain and clears stale browser state

## Architecture Notes

**Cloudflare TLS bypass:** claude.ai's API returns 403 when called directly with `fetch` or `curl` due to Cloudflare's TLS fingerprint detection. The tool uses headless Playwright/Chromium to first hit `claude.ai` (acquiring a clearance cookie), then injects the `sessionKey` cookie and navigates to the usage API.

**Browser state persistence:** After each successful fetch, `context.storageState()` saves all cookies, localStorage, and sessionStorage to `runtime/claude-usage/browser-state.json`. Subsequent fetches within 90 minutes load this state and skip the warm-up page, cutting fetch time from ~8–10s to ~2–4s.

**Atomic cache writes:** `--cache` mode writes to `.cache.json.tmp` then renames atomically, preventing partial reads by the statusline.

**Credential security:** Credentials are stored via macOS `security` CLI (`add-generic-password`) and retrieved with `find-generic-password -w`. They never touch the filesystem.

**sessionKey lifecycle:** The `sessionKey` cookie is a long-lived session token (~30-day validity). When it expires, the API returns `account_session_invalid` — the tool detects this, writes an error JSON to cache, and prompts to re-run setup.
