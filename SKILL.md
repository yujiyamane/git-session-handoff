---
name: git-session-handoff
description: >
  Git commit convention for session continuity — encodes session state
  (completed/in-progress/failed/next) into commit bodies, with a task management
  gate and 7-day age-out. Use at session start, before any git commit,
  at checkpoint or session end, when writing a commit message, or on
  handoff. Also use for session triage, commit formatting, or task promotion
  from git.
---

# Git Session Handoff

## Commit Message Format

Write the commit body to a temporary UTF-8 file and commit via `git commit -F <tmpfile>`
(or `git commit --allow-empty -F <tmpfile>` for empty commits). Never use inline `-m` for
multi-line bodies — shell argument parsing mangles multi-line strings and emoji characters.

```
<type>: <subject line, imperative mood, ≤72 chars>

✅ Completed:
- <concrete deliverable with function/file names>
- <test results with test names and PASS/FAIL>

🚧 In Progress:
- <partially done item with specific state>
- <what remains to finish>

❌ Failed:
- <approach that was tried>
- <why it failed — specific error or reason>
- <prevents future sessions from repeating the attempt>

➡️ Next:
- <first concrete action: function name, file path, or command>
- <subsequent actions in priority order>
- STILL OPEN from <YYYY-MM-DD>: <long-running item>

📤 Promoted to Task Manager:
✓ task: "<title>" (created: Tags=X, Due=YYYY-MM-DD)
```

## Type Prefixes (Conventional Commits)

| Type | Usage |
|---|---|
| `feat` | New functionality |
| `fix` | Bug fix |
| `refactor` | Code restructure without behaviour change |
| `chore` | Session start, checkpoint, config, maintenance |
| `style` | Formatting, CSS, layout, visual polish |
| `docs` | Documentation, spec updates |
| `test` | Test-only changes |

## Section Rules

| Section | Required? | When to omit | Max items |
|---|---|---|---|
| ✅ Completed | Always required | Never omit | 5 (group related items if more) |
| 🚧 In Progress | Include if applicable | Omit when nothing is partial | 3 |
| ❌ Failed | Include if applicable | Omit when nothing failed | 3 |
| ➡️ Next | Always required | Use "None — task complete." only when genuinely done | 5 |
| 📤 Promoted to Task Manager | Include when items promoted | Omit when no promotions | No limit |

The max-items rule keeps commit bodies within the ~1,000 token session-start budget.
If more than 5 items exist for ✅, group related items into a single line.

## Label Tags for Multi-Task Sessions

ALL items in ALL sections (✅, 🚧, ❌, ➡️) MUST have a 【label】 tag. No exceptions.

## Quality Criteria for ➡️ Next

Each item must be actionable without further context. Test: could a fresh session execute
this item by reading only the commit message?

| Quality | Example |
|---|---|
| ❌ Vague | "Continue working on the feature" |
| ❌ Ambiguous | "Fix the bug" |
| ✓ Actionable | "Implement `formatDuration_()` in `src/utils.js` — convert decimal hours to `Xh Ym`" |
| ✓ Actionable | "Add `testCompassEdgeCases` — verify `degreesToCompass_(360)` returns 'N' not undefined" |

## Special Commit Types

### Session Start Commit

No code changes. `--allow-empty` is mandatory.

    git commit --allow-empty -F <tmpfile>

Body:

    chore: session start — <primary goal>
    
    ✅ Completed:
    - Read git log -5: identified N open items from previous sessions
    - Decision: <priority order and rationale>
    
    ➡️ Next:
    - <first task to begin>

### Checkpoint Commit (mid-session)

Use `--allow-empty` if no file changes since last commit.

    chore: checkpoint — <context label>
    
    ✅ Completed:
    - <work done since last commit>
    
    🚧 In Progress:
    - <current state of active work>
    
    ➡️ Next:
    - <immediate next action in this session>

### Exploration / Read-Only Session Commit

Sessions producing no code changes still require a commit. `--allow-empty` is mandatory.

    chore: checkpoint — exploration session
    
    ✅ Completed:
    - Investigated <topic>: read <files/docs>, traced <call chain>
    - Key finding: <what was discovered>
    - Decision: <architectural or approach decision made>
    
    ❌ Failed:
    - <approaches considered and rejected, with reasons>
    
    ➡️ Next:
    - <concrete implementation action based on findings>

### Task Complete Commit

    feat: <completed feature>
    
    ✅ Completed:
    - <all deliverables>
    - <all tests passing>
    
    ➡️ Next:
    - None — task complete. <optional: monitoring note>

## Task Management Gate

### Gate Criteria

A ➡️ Next item is promoted to your task management system when **ANY** of the following
conditions are met:

**Standard gate — ALL three must be true:**
1. **Non-code action** OR **involves another person** OR **has a hard deadline**
2. **Cannot be resolved in the next coding session** (requires action outside the editor)
3. **Would be forgotten without a reminder** (not naturally encountered in code)

If any criterion is not met, the item stays in git only. When in doubt, stays in git.

**Age-out gate — regardless of standard criteria:**
4. Any ➡️ item marked `STILL OPEN from <date>` where `<date>` is **7 or more days ago**
   is promoted unconditionally. Active items use 🚧 In Progress, not STILL OPEN — the STILL
   OPEN marker signals dormancy, and 7 days of dormancy triggers automatic promotion.

### Creation Mechanism

1. Draft the commit body with all ✅🚧❌➡️ sections
2. Evaluate each ➡️ item against gate criteria, including age-out on all STILL OPEN items
3. For qualifying items, call your task management MCP tool to create the task
4. **Remove** the promoted item from ➡️ Next; record in `📤 Promoted to Task Manager:` block
5. Execute the git commit

**Removal contract:** Once an item appears in `📤 Promoted to Task Manager:`, it is no longer
in ➡️ Next and will never be re-evaluated. This prevents duplicate task creation.

If the task management MCP is unavailable, the commit proceeds without the reference.
Annotate the item with `(Task: deferred — MCP unavailable)` — it remains in ➡️ for the
next session's gate evaluation.

### Gate Examples

| ➡️ Item | Gate Result | Reasoning |
|---|---|---|
| `formatDuration_()` implementation | Git only | Pure code, next session |
| Share setup steps with team member | **✓ Task** | Other person, non-code |
| Monitor system for 3 days | **✓ Task** | Deadline, would forget |
| Research API rate limits | Git only | Research for next code session |
| Update spec doc | Git only | Done alongside code changes |
| Ask colleague about architecture | **✓ Task** | Other person, non-code |
| Deploy and verify output | Git only | Immediate dev task |
| Book team meeting by Friday | **✓ Task** | Other people, deadline, non-code |
| STILL OPEN from 2026-05-28: investigate flaky test | **✓ Task** | Age-out: 7+ days dormant |

## Session Start git log Command

Run at every session start to read previous context:

    git log -5 --invert-grep --extended-regexp --grep="^chore: session start" --format="=== %h %s%n%b"

- `-5` limits to 5 most recent non-session-start commits
- `--invert-grep` filters out `chore: session start` commits (retained in history for
  time tracking, excluded from context read to keep the window useful)
- `--extended-regexp --grep="^chore: session start"` anchors to subject line prefix only
- `=== %h %s` separator makes multi-commit output parseable

Read the most recent ➡️ Next, then create the session start commit immediately.
