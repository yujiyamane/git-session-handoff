---
description: "Close session: commit structured handoff, evaluate task gate, write handoff note, push"
allowed-tools:
  - Bash(git add:*)
  - Bash(git status:*)
  - Bash(git commit:*)
  - Bash(git diff:*)
  - Bash(git push:*)
  - YOUR_MCP_TOOL_NAME
---

NOTE: Replace YOUR_MCP_TOOL_NAME in the allowed-tools above with the canonical `mcp__…`
name for your task management tool (e.g. `mcp__claude_ai_Notion__notion-create-pages`).

Based ONLY on this conversation session:

**Call 1 — Assess and stage (one bash call):**
`git diff --stat HEAD && git status && git add -A`

Using the output above, draft a commit message following the git-session-handoff skill:
- Subject: conventional commit type (feat/fix/chore/refactor/style/docs/test), imperative mood, ≤72 chars
- ✅ Completed: what was delivered, with function/file names and test results (max 5)
- 🚧 In Progress: partial work with specific state (max 3, omit if none)
- ❌ Failed: what was tried and WHY it failed (max 3, omit if none)
- ➡️ Next: actionable next steps a fresh session can execute (max 5)
- STILL OPEN from <YYYY-MM-DD>: carry forward any dormant items

Task management gate (MCP calls only when criteria met — usually 0):
- Standard gate: non-code action + can't resolve next session + would forget → promote
- Age-out gate: STILL OPEN dated 7+ days ago → promote unconditionally
- For qualifying items: call task management MCP, remove item from ➡️, record in 📤 Promoted to Task Manager block

**Call 2 — Commit, handoff, push (one bash call):**
Chain all of the following in a single bash call:
- Write commit message to `/tmp/gsh_commit.txt`
- If heuristics met (3+ files changed OR ❌ present OR 🚧 present): write `~/.claude/handoff.md` in UTF-8
- `git commit -F /tmp/gsh_commit.txt` (use `--allow-empty` if exploration-only)
- `git push`
- `rm -f /tmp/gsh_commit.txt`
- `echo "✅ Session committed and pushed. Close or open a new session."`

Handoff note format (write to `~/.claude/handoff.md`):
    📝 Handoff Note
    Current state:
    - <what I was working on and where I got to>
    Key decisions:
    - <why approach X was chosen over Y>
    Watch out:
    - <traps, edge cases, things that almost worked>
    Key files:
    - <file:line references for next session>

Rules:
- ALL items in ALL sections (✅, 🚧, ❌, ➡️) MUST have a 【label】 tag. No exceptions.
- Every ➡️ item must be executable by a fresh session from the commit message alone.
- ✅ must reference actual function names, file paths, or test names.
- ❌ must include WHY it failed, not just what was tried.
