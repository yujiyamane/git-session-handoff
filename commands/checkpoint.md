---
description: "Mid-session checkpoint: capture current state in a structured commit"
allowed-tools:
  - Bash(git add:*)
  - Bash(git status:*)
  - Bash(git commit:*)
  - Bash(git diff:*)
---

Check git status for staged/unstaged changes.
Stage relevant changes (or use `--allow-empty` if no changes since last commit).
Write commit message to a temporary UTF-8 file.
Create checkpoint commit (`git commit -F <tmpfile>` or `--allow-empty -F <tmpfile>`)
with structured body per git-session-handoff skill.
Include 🚧 In Progress for any active work.
