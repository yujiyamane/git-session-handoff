---
description: "Start a new session: read git log, triage open items, create session start commit"
allowed-tools:
  - Bash(git log:*)
  - Bash(git commit:*)
---

Run `git log -5 --invert-grep --extended-regexp --grep="^chore: session start" --format="=== %h %s%n%b"`.
Identify all ➡️ Next items from the most recent commit.
Identify any STILL OPEN items — check dates for 7-day age-out.
Create a triage plan: prioritise quick wins first, then main goal.
Write commit message to a temporary UTF-8 file.
Execute: `git commit --allow-empty -F <tmpfile>` with structured body
per git-session-handoff skill.
