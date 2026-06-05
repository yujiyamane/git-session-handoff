# git-session-handoff

Zero-overhead session continuity for Claude Code — encodes session state into git commit bodies.

## What it is

Each Claude Code session starts with no memory of what the previous session accomplished.
`git-session-handoff` solves this by encoding session state directly into git commit message
bodies using a structured convention. At session start, a single `git log` command recovers
all open items, failed approaches, and next actions from the previous session in under 30
seconds — no hooks, no extra files, no external dependencies.

The convention stores four types of information in every commit:

- **✅ Completed** — what was delivered (with test results)
- **🚧 In Progress** — partial work and its current state
- **❌ Failed** — dead ends that future sessions should not repeat
- **➡️ Next** — actionable next steps (each executable by a fresh session from the commit alone)

A task management gate promotes long-running or non-code items from ➡️ Next to your external
task manager (via MCP), with a 7-day age-out that prevents items from silently rotting in git.

**Performance:** ~300–1,000 tokens at session start (one `git log` call). Zero per-turn overhead.

## Install

1. Copy `SKILL.md` to `~/.claude/skills/git-session-handoff/SKILL.md`
2. Copy `examples/worked-examples.md` to `~/.claude/skills/git-session-handoff/examples/worked-examples.md`
3. Copy the two files in `commands/` to `~/.claude/commands/`
4. Append the following block to `~/.claude/CLAUDE.md`:

```
## Git Session Handoff

REQUIRED first action of every session:

0. Run in one call:
   - `git status`
   - `cat ~/.claude/handoff.md 2>/dev/null`
   - `git log -5 --invert-grep --extended-regexp --grep="^chore: session start" --format="=== %h %s%n%b"`

1. If uncommitted changes detected: show diff summary and ask
   "⚠️ Uncommitted changes detected. Commit these? (Y/n)"

2. If handoff.md non-empty and <24h old: display contents, then delete the file.
   If >24h old: skip with "⚠️ Stale handoff note — skipping."

3. Display Session Brief from git log output.

4. Start working. No session start commit.

All commits: follow the git-session-handoff skill for format and task gate rules.
```

5. Edit `commands/close.md`: replace `YOUR_MCP_TOOL_NAME` with your task management
   MCP tool (e.g. `mcp__claude_ai_Notion__notion-create-pages`).
6. Run `git config --global i18n.commitEncoding utf-8` (required for emoji integrity).

## Commit Format

```
feat: add weather API integration to daily digest

✅ Completed:
- Open-Meteo API integration (temperature, wind speed, conditions)
- 3 tests PASS: testWeatherFetch, testWeatherHtml, testNoDataFallback

🚧 In Progress:
- Humidity data API call written but not wired to HTML template

❌ Failed:
- WeatherAPI.com — requires paid key, abandoned after 20 min research

➡️ Next:
- Wire humidity to template (function exists: formatHumiditySection_)
- Fix 360/0 boundary in degreesToCompass_(), add testCompassEdgeCases
- STILL OPEN from 2026-06-03: score rounding bug (low priority)
```

See [examples/worked-examples.md](examples/worked-examples.md) for full realistic examples.

## Commands

| Command | Purpose |
|---|---|
| `/checkpoint` | Mid-session commit capturing current state |
| `/close` | Final commit with full task management gate evaluation + handoff note (recommended at session close) |

## Handoff Notes

For complex sessions, `/close` writes `~/.claude/handoff.md` — an ephemeral note carrying
nuance that doesn't fit in a commit body (current mental model, key file locations, traps
to avoid, reasoning behind decisions).

**Lifecycle:** Written by `/close`, read and atomically cleared by the next session start.
Skipped automatically if the file is older than 24 hours (handles crashed sessions).

**When it's written:** Claude applies three heuristics at close time — if any are true, the
handoff note is written; otherwise it's skipped and the commit body is sufficient:
- Session touched 3+ files
- ❌ Failed items present (active debugging session)
- 🚧 In Progress items present (partial execution)

**Format:**

    📝 Handoff Note
    Current state:   — where you got to
    Key decisions:   — why X over Y
    Watch out:       — traps and edge cases
    Key files:       — file:line refs for next session

The file lives in `~/.claude/` (outside any project repo — no `.gitignore` needed).
Encoding: UTF-8.

## How it works

```
Session N                          Session N+1
─────────                          ──────────
START                              START
  │                                  │
  ├─ git log -5 (filtered)           ├─ git log -5 (filtered)
  │    └─ Read most recent ➡️ Next   │    └─ Read most recent ➡️ Next
  │                                  │
  ├─ ... work ...                    ├─ ... work ...
  │                                  │
  ├─ feat/fix/refactor commit        │
  │    └─ Structured body            │
  │       (✅ 🚧 ❌ ➡️)              │
  │                                  │
  └─ /close                          │
       └─ Final commit + gate eval   │
```

The `--invert-grep` filter excludes legacy `chore: session start` commits from the context
window. Session start commits are no longer created; the filter exists for backward
compatibility with older history.

## Task Management Gate

➡️ Next items are promoted to your external task manager when they meet the **standard gate**
(non-code action + can't resolve next session + would forget) OR the **age-out gate** (marked
`STILL OPEN from <date>` and that date is 7+ days ago).

Items that qualify are removed from ➡️ Next and recorded in `📤 Promoted to Task Manager:`.
This removal contract prevents duplicate task creation across sessions.

## Dev Time Tracking

Git commit timestamps provide session boundary data with no extra instrumentation:

```bash
# Yesterday's sessions
git log --format="%aI %s" --since="1 day ago" --until="today"

# This week's commit types
git log --format="%s" --since="last monday" | cut -d: -f1 | sort | uniq -c

# Search all failed approaches
git log --all --grep="❌" --format="%h %s%n%b"
```

Git commit timestamps provide the authoritative session boundaries.

## License

MIT © 2026 Yuji Yamane
