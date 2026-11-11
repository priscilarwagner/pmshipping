---
name: weekly-wrap
description: "Weekly review + compound learnings — review progress, surface blockers, distill insights into Memory"
---

# Weekly Wrap

Run a structured weekly review, produce a shareable status update, then distill learnings into compounded knowledge.

## Instructions

When the user invokes `/weekly-wrap`, run all steps in sequence **without waiting for prompts between them**. Produce the final shareable update, learnings, and next-week focus in a single pass. Save everything, then confirm.

**Do NOT stop to ask for confirmation between steps.** The user will correct anything after seeing the full output.

---

### Step 1: Gather the Week's Raw Material

Pull everything from the target week before synthesising anything.

1. **Weekly scratchpad** — read `Tasks/Week-YYYY-WNN.md` (ISO week number)
   - Count ticked `[x]` vs unticked `[ ]` items across all daily sections
   - Extract all free-form notes, decisions, and meeting entries from Notes sections
   - Note which days had entries
2. **Meeting notes** — scan `Context/Meeting Notes/` for files dated this week
   - Extract: attendee names, decisions made, action items with owners, blockers raised
3. **Task files** — read `Tasks/` root for `status: d` (done this week) and `status: b` (blocked). Week scratchpads (`Week-YYYY-WNN.md`) are not tasks — skip them
   - Also scan `Tasks/Backlog/` to assess initiative-level progress beyond individual task completions — flag any track that has had no slice activity this week
   - Also flag `status: s` with no progress log entry in the last 7 days (stalled)
4. **Goals** — read `GOALS.md` for quarterly objectives and top 3 priorities
5. **Prior wrap** — read the most recent file in `Context/Progress Updates/` to compare trajectory
6. **Connected tools** — if Linear and Gmail are available, pull the week's key activity:
   - Linear: issues completed or moved this week (`list_issues`, `assignee: "me"`, `updatedAt: -P7D`), project updates posted
   - Gmail (`in:inbox newer_than:7d -category:promotions -category:updates -category:social -category:forums` and `in:sent newer_than:7d`): decisions in threads, action items, blockers, escalations
   - If tools are unavailable, note this and proceed — don't block the wrap

---

### Step 2: Internal Review (not shown to user)

Before writing the status update, do this analysis privately:

- Map completed work to goals — which goals moved, which didn't
- Identify the 4-6 most significant things that happened (decisions, deliverables, meetings, shifts)
- List every blocker and stalled item with owner and how long it's been stuck
- Pull specific names from meeting notes for action items
- If `Context/Memory/bias.md` exists, check against bias patterns — did any show up this week?

---

### Step 3: Write the Weekly Status Update

**Produce ONLY the short, shareable version directly.** No long draft first. Target ~200 words. Use simple language.

Use this structure:

```
# Weekly Update CW[XX] — [DD–DD Mon YYYY]

**In short:**
[2-3 sentences: what happened, what succeeded, what's blocked. Plain English, data where useful.]

## What happened
- [Thing that moved — who was involved, what was decided]
- [Thing that moved — who was involved, what was decided]
- [Thing that moved — who was involved, what was decided]
(4-5 bullets max. Only things that actually moved.)

## What's next
- [Next action — who owns it, rough timing]
- [Next action — who owns it, rough timing]
- [Next action — who owns it, rough timing]
```

**Tone rules:**
- Simple, direct language — as if explaining to a smart colleague over coffee
- Data where it adds clarity, not decoration
- Specific names, not "the team" or "stakeholders"
- Optimistic framing — what moved, not what failed

---

### Step 4: Write the Full Wrap (internal detail, appended below the shareable update)

After the shareable update, append these sections for the saved wrap file:

```
## Status Check
| Initiative | Status | Notes |
|------------|--------|-------|
| [goal/initiative] | On track / At risk / Blocked / Paused | [short note] |

## CW[XX+1] Focus

**Must do:**
1. [item] — Goal: [which goal]
2. ...

**Should do:**
1. [item]
2. ...

**Watch:**
- [anything at risk]

## Learnings (saved to Memory)
- [3-5 bullet learnings — insights, patterns, bias flags]

## System Health
- [files missing descriptions, index freshness]
```

Flag if any active quarterly objective has had zero activity two weeks running.

---

### Step 5: Compound Learnings into Memory

Distill the week into durable knowledge for future sessions.

1. Read `Context/Memory/learnings.md` — avoid repeating what's already captured
2. Extract 3-5 learnings:
   - **Insights**: what worked, what didn't, and why (evidence-backed)
   - **Patterns**: recurring themes across tasks, blockers, or decisions
   - **Bias flags**: if a bias pattern appeared this week, name it and note context
   - If something keeps recurring across multiple wraps, promote it to a standing principle
3. **Save directly** — append a new dated section to `Context/Memory/learnings.md` (3-5 bullets max)
4. Do NOT ask for confirmation — save and show what was added in the output

### Compounding Principle

Each wrap should make the next one better:
- Reference prior learnings when assessing this week
- Promote repeated patterns into standing principles
- Prune stale learnings that no longer apply
- If `learnings.md` grows past ~50 entries, suggest consolidating into higher-level principles

---

### Step 6: System Health & Kanban Sync

1. Run a quick check:
   - Count active task files (Tasks/ root, not Done/) missing `description` field
   - Report in the wrap: "System health: X files need enrichment."

2. Kanban and Done sync:
   - Move completed tasks (`status: d`) from "In Progress" to "Done" in `Weekly Kanban.md` and mark them `[x]`
   - Move each completed task file from `Tasks/` to `Tasks/Done/` (wiki-links still resolve in Obsidian)

3. **Promotion check** — for each completed task, check if it produced promotable outputs:
   - Strategy docs, decision records → flag for `Context/Document Hub/`
   - Frameworks, playbooks, templates → flag for `Context/Memory/Reference/`
   - Report: "Promotable outputs: [list with suggested destinations]" — let the user confirm before promoting

---

### Step 7: Save the Wrap

Save the full wrap (shareable update + internal detail sections) to `Context/Progress Updates/`.

Use a consistent naming convention: `CW[XX]-YYYY.md` (e.g. `CW11-2026.md`).

If a file for this CW already exists, append the new wrap at the top (most recent first) rather than overwriting.

Confirm: *"Saved to `Context/Progress Updates/CW[XX]-YYYY.md`. Learnings appended to Memory."*

---

## Key Principle: No Mid-Flow Prompts

The entire wrap — shareable update, internal detail, learnings save, system health, and file save — runs as one uninterrupted pass. The user reviews the complete output and provides corrections after, not during. This eliminates back-and-forth rounds.

If any external tool is unavailable, note it and continue. Never block the wrap waiting for a tool.
