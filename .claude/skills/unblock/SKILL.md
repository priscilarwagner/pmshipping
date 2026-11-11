---
name: unblock
description: "Diagnose a stalled task and suggest the smallest next action to get it moving"
---

# Unblock

> **Tools:** steps call role operations (`pm.*`, `email.*`, `calendar.*`, `docs.*`, `files.*`, `comms.*`). Resolve each through `Context/agent-config.md` → **Tool Stack** and the adapter it names (`.claude/adapters/README.md`). A role set to `none` → skip that step.

Diagnose why a task has stalled and suggest the smallest possible next action to get it moving again.

## Instructions

When the user invokes `/unblock [task name]`, run through all steps in sequence.

If no task name is provided, scan `Tasks/` for the most obviously stalled item (in-progress with no recent progress log entry) and suggest it. Also check project management: high/urgent items from `pm.my_items(backlog)` and `pm.my_items(todo)`, and `pm.my_items(started)` not updated for 7+ days.

### Step 1: Read the Task

1. Find and read the task file in `Tasks/` — or, if the user named a project-management item (e.g. `ENG-123`), read it with `pm.item` and `pm.item_comments`
2. Note: status, priority, created date, last progress log entry, sub-tasks (checked vs unchecked)
3. Calculate how long it's been stalled (days since last progress log entry or creation if no log)

### Step 2: Diagnose the Stall

Check for common stall patterns:

| Pattern | Signal | Likely Cause |
|---|---|---|
| Empty sub-tasks | `- [ ]` with no text | Task was never broken down |
| No progress log | Missing or only "Created" entry | Never actually started |
| Blocked dependency | References another task or person | Waiting on someone else |
| Too big | 5+ unchecked sub-tasks, broad title | Needs to be split |
| Missing context | No goal reference, vague description | Unclear why it matters |
| Unclear next action | Sub-tasks are vague or outcome-level | Needs concrete first step |

### Step 3: Search for Context

1. Search `Context/` for files related to the task topic
2. Search `Context/Meeting Notes/` for recent discussions about this work
3. Search `email.search`, `pm.search` and `comms.search` (where configured) for this topic
4. Check `GOALS.md` — is this task still aligned to an active goal? If not, flag it.

### Step 4: Suggest the Unblock

Based on the diagnosis, suggest **one** concrete next action. Not a plan — a single thing the user can do in the next 15 minutes:

- **If never broken down:** "The first sub-task should be: [specific action]"
- **If waiting on someone:** "Send [person] this message: [draft a short nudge]"
- **If too big:** "Split this into [2-3 smaller tasks]. Start with: [the first one]"
- **If unclear why it matters:** "This doesn't connect to any current goal. Archive it or tell me why it matters."
- **If unclear next action:** "The concrete next step is: [specific action with a verb]"

### Step 5: Offer to Update

Ask: "Want me to update the task file with this next action and a progress log entry?"

If yes:
1. Add the suggested next action as the first unchecked sub-task
2. Add a progress log entry: `YYYY-MM-DD: Unblocked — [diagnosis and next step]`
3. Update status to `s` (started) if it was `n` (not started)

## Output Format

Keep it short. The diagnosis should be 1-2 sentences. The suggested action should be one concrete step, not a plan. The goal is momentum, not perfection.
