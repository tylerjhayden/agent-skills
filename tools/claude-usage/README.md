# claude-usage

> Fetches real-time claude.ai session and weekly usage limits using headless Playwright to bypass Cloudflare TLS fingerprinting, then displays progress bars per model tier. Includes a background poller that keeps a cache file fresh for statusline integration.
>
> Part of [agent-toolkit](https://github.com/tylerjhayden/agent-toolkit)

## Installation

```bash
# Clone the repo (or download just this directory)
git clone https://github.com/tylerjhayden/agent-toolkit

# Symlink the script onto your PATH
ln -s /path/to/agent-toolkit/tools/claude-usage/claude-usage.ts ~/bin/claude-usage
chmod +x ~/bin/claude-usage

# Install Playwright's Chromium (one-time)
bunx playwright install chromium
```

Both you and Claude Code can invoke `claude-usage` directly once it's on your PATH.

## Requirements

- macOS
- Bun (bun.sh)
- Playwright for Bun (`bunx playwright install chromium`)
- macOS Keychain (security CLI)

## Why Playwright?

claude.ai's API returns 403 when called directly with `fetch` or `curl`. Cloudflare detects standard HTTP clients by their TLS fingerprint before the request even reaches the server. This tool uses headless Chromium (via Playwright) to acquire a Cloudflare clearance cookie, then injects your `sessionKey` cookie and fetches the usage API. There's no other reliable way to reach it programmatically.

After a successful fetch, the browser state (cookies, localStorage) is saved to `runtime/claude-usage/browser-state.json`. Subsequent fetches within 90 minutes load that state and skip the warm-up page — cutting fetch time from ~8–10s to ~2–4s.

## What It Does

- **Session usage:** 5-hour rolling window utilization and reset time
- **Weekly usage:** 7-day aggregate across all models, Sonnet, and Opus tiers
- **Human display:** Progress bars with percentage and countdown to reset
- **JSON output:** Raw API response for scripting or inspection
- **Cache mode:** Writes structured JSON to `runtime/claude-usage/cache.json` for statusline integration
- **Background poller:** Session-aware LaunchAgent — only fetches when Claude Code is running, adapts interval based on utilization

## First-Time Setup

```bash
claude-usage setup
```

This prompts for two values and stores them in macOS Keychain (never on disk):

**`sessionKey`** — your claude.ai session cookie:
1. Open [claude.ai](https://claude.ai) in Chrome
2. Open DevTools → Application → Cookies → `claude.ai`
3. Find the cookie named `sessionKey` and copy its value

**`orgId`** — your organization UUID:
1. Open [claude.ai/settings/usage](https://claude.ai/settings/usage)
2. Open DevTools → Network → reload the page
3. Find the `usage` API call — the org UUID is in the URL path

You'll re-run setup approximately every 30 days when the `sessionKey` expires.

## Usage

```bash
# Check usage (human-readable progress bars)
claude-usage

# Raw JSON output (full API response)
claude-usage --json

# Write cache file (for poller / statusline)
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

## Statusline Integration

Claude Code supports a custom statusline via `.claude/settings.json`. Point it at a shell script that reads `cache.json` and outputs a formatted string:

```json
// .claude/settings.json
{
  "statusLine": {
    "type": "command",
    "command": "/path/to/your/statusline-script.sh",
    "padding": 0
  }
}
```

The poller writes `runtime/claude-usage/cache.json` after every fetch. Your statusline script reads it and formats the output however you like. Example of what integrated output looks like:

```
Sonnet 4.5 | main | 27% ctx | 25% 22m↺       # fresh — session utilization + time to reset
Sonnet 4.5 | main | 27% ctx | ~25% 22m↺      # stale — cache older than 15 min
Sonnet 4.5 | main | 27% ctx | ⚠ usage:expired # session key needs refresh
Sonnet 4.5 | main | 27% ctx                   # no cache yet (poller hasn't run)
```

`cache.json` structure:
```json
{
  "fetched_at": "2026-02-17T19:07:01Z",
  "five_hour": {
    "utilization": 37,
    "resets_at": "2026-02-17T21:30:00Z"
  },
  "seven_day": {
    "all_models": { "utilization": 28 },
    "sonnet": { "utilization": 19 }
  }
}
```

## Background Poller

> **Customization required:** Replace `my-project` with your project directory name and `com.myproject` with your own reverse-domain identifier (e.g., `com.yourname`) throughout the poller script and plist before installing.

The LaunchAgent fires every 2 minutes but exits in <100ms when Claude Code isn't running. When active, it adapts fetch frequency to utilization:

- **< 80%** → fetch every ~10 min
- **≥ 80%** → fetch every ~2 min
- **Session window expired** → immediate force-fetch

```bash
# Load (one-time setup)
launchctl load ~/Library/LaunchAgents/com.myproject.claude-usage-poller.plist

# Trigger manually
launchctl start com.myproject.claude-usage-poller

# Watch operational log
tail -f ~/my-project/logs/claude-usage-poller.log

# Disable
launchctl unload ~/Library/LaunchAgents/com.myproject.claude-usage-poller.plist
```

## Error Recovery — Session Expiry (~30-day cycle)

When `sessionKey` expires, the failure surfaces at every level:

1. **Poller error log** (`logs/claude-usage-poller-error.log`) shows:
   ```
   [SESSION EXPIRED] Run: claude-usage setup
   Steps: open https://claude.ai/settings/usage → DevTools → Application
          → Cookies → claude.ai → copy sessionKey value
   ```
2. **Cache file** contains `"error": "session_expired"`
3. **Statusline** shows `⚠ usage:expired`
4. **Fix**: `claude-usage setup` — re-stores fresh sessionKey in Keychain

## Architecture Notes

**Cloudflare TLS bypass:** claude.ai's API returns 403 when called directly with `fetch` or `curl` due to Cloudflare's TLS fingerprint detection. The tool uses headless Playwright/Chromium to acquire a clearance cookie, then injects the `sessionKey` cookie and navigates to the usage API.

**Browser state persistence:** After each successful fetch, Playwright's `storageState()` saves cookies and localStorage to `runtime/claude-usage/browser-state.json`. Subsequent fetches within 90 minutes load this state and skip the warm-up page, cutting fetch time from ~8–10s to ~2–4s.

**Atomic cache writes:** `--cache` mode writes to `.cache.json.tmp` then renames atomically, preventing partial reads by statusline scripts.

**Credential security:** Credentials are stored via macOS `security` CLI (`add-generic-password`) and retrieved with `find-generic-password -w`. They never touch the filesystem.

## License

MIT — see [LICENSE](../../LICENSE).
