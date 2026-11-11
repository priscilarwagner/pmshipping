---
name: backlog
description: "Triage the BACKLOG.md inbox — extract items, enrich with context, create task files, clear the backlog"
---

# Process Backlog

Triage the raw capture inbox (`BACKLOG.md`) into structured, goal-aligned task files.

## Instructions

When the user invokes `/backlog` (or says "clear my backlog", "process backlog", etc.), run through all steps in sequence.

### Step 1: Read the Backlog

1. Read `BACKLOG.md` and extract every actionable item
2. If the backlog is empty, tell the user and stop

### Step 2: Enrich with Context

1. For each item, search `Context/` for relevant context — matching keywords, project names, or dates
2. Check `Context/Memory/` for user preferences that affect prioritization (e.g., standing priorities, delegation patterns)
3. Reference `GOALS.md` to identify which goal each item supports

### Step 3: Clarify Ambiguous Items

1. If any item lacks context, priority, or a clear next step — **stop and ask** the user for clarification before creating a task
2. Batch all clarification questions into a single prompt rather than asking one at a time
3. Suggest a best-guess priority and goal alignment for each unclear item so the user can confirm or correct

### Step 4: Deduplicate

1. Read existing task files in `Tasks/` and `Tasks/Backlog/` (check titles and content). Also check Linear (`list_issues` with `query`) for the same work
2. Flag any backlog item that duplicates or overlaps with an existing task
3. For duplicates, suggest updating the existing task instead of creating a new one

### Step 5: Create Task Files

Decide where each item belongs:

- **(a) New initiative/track**: if the item is a multi-step initiative that needs strategic context, decisions, and frameworks — create a context file in `Tasks/Backlog/` with background and decisions, then create initial actionable slices in `Tasks/` root
- **(b) Standalone task**: if it's a single actionable item — create directly in `Tasks/` root

Create each task using this template:

```yaml
---
title: [Actionable task name]
description: "[~150 chars — retrieval filter for what makes this task distinctive]"
type: task
category: [technical|outreach|research|writing|content|admin|personal|other]
topics: ["topic-1", "topic-2"]  # relevant tags for this task
priority: [P0|P1|P2|P3]
status: n
created_date: [YYYY-MM-DD]
due_date: [YYYY-MM-DD]  # optional
resource_refs:
  - Context/relevant-file.md
---

# [Task name]

## Context
[Tie to goals — cite the specific heading or bullet from GOALS.md]

## Next Actions
- [ ] Step one
- [ ] Step two

## Progress Log
- YYYY-MM-DD: Created from backlog.
```

Each task **must**:
- Reference a goal in its Context section. If no goal fits, flag it and ask whether to create a new goal entry or clarify why the work matters.
- Have a `description` (~150 chars, written so a future session can decide whether to open the file from the description alone) and a `topics` field.

### Step 6: Summarize and Clear

1. Present a concise summary table of all new/updated tasks:

| Task | Priority | Goal | Action |
|------|----------|------|--------|
| [name] | P1 | [goal ref] | Created / Updated existing |

2. Flag any items that don't align to a current goal
3. **Kanban sync**: For each new task, append a Kanban entry to `Weekly Kanban.md` under the "Backlog" column: `- [ ] [[Task Name]] — one-line summary #priority`
4. Ask: "Ready to clear the backlog?"
5. On confirmation, clear `BACKLOG.md` back to its default state:

```markdown
# Backlog

Drop raw notes or todos here. Say `/backlog` when you're ready for triage.
```

### Step 7: Compound-on-Touch (silent)

After processing and clearing the backlog:

1. **Recurring items** — if a similar item has appeared in the backlog 3+ times (check `Context/Memory/learnings.md` and prior task files), flag the pattern and suggest a standing task or process change. Append to learnings:
   ```
   ## [date]
   - [pattern — e.g., "Third time 'update stakeholders on X' appeared in backlog — consider a recurring weekly task"]
   ```
2. **Missing metadata** — enrich any task files you touched that lack `description`

Do NOT announce these fixes. Just do them.

---

## Output Format

Keep it scannable — use the summary table, not walls of text. Batch all clarification questions into one round to avoid back-and-forth.
