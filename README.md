# claude-live-usage

A live terminal dashboard for [Claude Code](https://claude.ai/code) that shows your rate-limit consumption and per-model token/cost breakdown in real time.

```
 ⚡ Claude Usage                                      14:32:07 UTC  q=quit
──────────────────────────────────────────────────────────────────────────
Rate Limits
  5-hour    [████████████████░░░░░░░░░░░░░░░░░░░░░░]  41.2%  resets 3h 12m
  weekly    [████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░]   9.8%  resets 5d 04h

──────────────────────────────────────────────────────────────────────────
This Month  March 2026
  Model                          Input   Output  Cache↓  Cost
  sonnet-4-6-20251101            12.4M    1.2M    8.3M   $45.2300
  opus-4-6                        2.1M  312.4K    1.5M   $38.1200
  TOTAL                          14.5M    1.5M    9.8M   $83.3500
```

## What it installs

| File | Location | Purpose |
|------|----------|---------|
| `usage` | `~/.local/bin/usage` | Curses TUI dashboard |
| `usage-statusline` | `~/.local/bin/usage-statusline` | Claude Code status-line hook that caches rate-limit data |

The installer also patches `~/.claude/settings.json` to register `usage-statusline` as the `statusLine` command, so Claude Code feeds it rate-limit data after every turn.

## Requirements

- **Claude Code** installed and configured (`~/.claude/` must exist)
- **Python 3.7+** (no third-party packages required — stdlib only)
- A terminal with 256-color support

## Install

```bash
bash install-claude-usage
```

Then open a new terminal (or `source ~/.bashrc` / `source ~/.zshrc`) and run:

```bash
usage
```

Rate-limit bars will populate after your next Claude Code interaction.

## Usage

| Key | Action |
|-----|--------|
| `q` / `Q` / `Esc` | Quit |

The dashboard refreshes every 0.5 seconds.

### Rate limit bars

- **Green** — below 60% used
- **Yellow** — 60–85% used
- **Red** — above 85% used

### Token table

Reads all session JSONL files under `~/.claude/projects/` and aggregates token counts and estimated costs for the current calendar month.

Pricing is baked in for the current Claude model lineup (Opus 4, Sonnet 4, Haiku 4, and the Claude 3.x series). Unknown models fall back to Sonnet 4 pricing.

## Uninstall

```bash
rm ~/.local/bin/usage ~/.local/bin/usage-statusline
```

Then remove the `statusLine` entry from `~/.claude/settings.json`.

## How it works

```
Claude Code turn
     │
     ▼
usage-statusline (statusLine hook)
     │  reads rate_limits from stdin JSON
     ▼
~/.claude/rate-limits-cache.json
     │
     ▼
usage (curses dashboard)
     │  also reads ~/.claude/projects/**/*.jsonl
     │  for token/cost totals
     ▼
terminal
```

Claude Code invokes the `statusLine` command after each turn, passing a JSON blob on stdin. `usage-statusline` extracts the `rate_limits` field and writes it to a cache file. The `usage` dashboard reads that cache file plus the session logs to render everything.

## License

MIT
