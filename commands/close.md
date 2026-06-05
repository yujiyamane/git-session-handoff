---
description: "Close session: commit structured handoff, evaluate task gate, push, then close or open a new session"
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

Stage all remaining changes (or use `--allow-empty` if exploration-only session).
Draft commit body with all ✅🚧❌➡️ sections per git-session-handoff skill.
Evaluate each ➡️ item against the task management gate criteria:
  - Standard gate: non-code action + can't resolve next session + would forget
  - Age-out gate: STILL OPEN dated 7+ days ago
For qualifying items:
  1. Call task management MCP to create the task
  2. Remove item from ➡️ Next
  3. Record in 📤 Promoted to Task Manager block
Write commit message to a temporary UTF-8 file.
Execute `git commit -F <tmpfile>` (or `--allow-empty` if no staged changes).
Execute `git push`.
Output: "✅ Session committed and pushed. Close or open a new session."

Rules:
- ALL items in ALL sections (✅, 🚧, ❌, ➡️) MUST have a 【label】 tag. No exceptions.
- Every ➡️ item must be executable by a fresh session from the commit message alone.
- ✅ must reference actual function names, file paths, or test names.
- ❌ must include WHY it failed, not just what was tried.
