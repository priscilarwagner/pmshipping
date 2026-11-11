---
name: 121
description: "Prepare for a 1:1 — pull context from goals, meeting notes, email, and tasks to generate talking points"
---

# 1:1 Prep

> **Tools:** steps call role operations (`pm.*`, `email.*`, `calendar.*`, `docs.*`, `files.*`, `comms.*`). Resolve each through `Context/agent-config.md` → **Tool Stack** and the adapter it names (`.claude/adapters/README.md`). A role set to `none` → skip that step.

Prepare focused talking points for a 1:1 meeting with a specific person.

## Instructions

When the user invokes `/121 [person name]`, run through all steps in sequence.

If no person name is provided, ask: "Who's the 1:1 with?"

### Step 1: Gather Context on This Person

1. Check `Context/121s/[Person Name].md` — if it exists, read the open loops and recent session log entries first (this is the richest source)
2. Search `Context/Meeting Notes/` for recent notes mentioning this person
3. Search `Context/Document Hub/` for shared docs, PRDs, or projects involving them
4. Check `GOALS.md` — identify which goals this person is connected to (e.g., key relationships, team members, stakeholders)
5. Check `Context/Memory/` for any stored preferences or notes about working with this person
6. `calendar.next_with(person, 14)` — note the date and any agenda in the description
7. `email.with(person, 30)` — recent threads with them

### Step 2: Review Your Recent Work

1. Scan `Tasks/` for active tasks this person is involved in or would care about
2. Check `Context/Progress Updates/` for the most recent weekly wrap — what moved, what's blocked
3. Note any updates, decisions, or completions worth sharing

### Step 3: Check for Open Loops

1. Search meeting notes for any action items assigned to you from previous 1:1s with this person
2. Search recent email threads with them for any unresolved asks or commitments
3. Flag anything unresolved — these go into the output

### Step 4: Generate Talking Points

Output four sections using the format below. Each section should have 3-5 short bullets max. The user will copy-paste a subset directly into their 1:1 doc, so every bullet must be self-contained and useful on its own.

**Sections:**

1. **Discuss** — Topics to raise: open loops, updates to share, things to demo or walk through. Lead with context ("Project X scaling: ...") not naked questions.
2. **Ask** — Direct questions where you need their input, perspective, or a decision. Frame each as a question or a position + question.
3. **Strategic** — Bigger-picture alignment topics: shared goals, org design, opportunities to explore together. Career/development if this is a direct report.
4. **Close the loop** — Unresolved action items from previous meetings or email threads that need a status check or handoff.

### Step 5: Update Ongoing Person Doc

Maintain a running relationship doc at `Context/121s/[Person Name].md`. This accumulates context across sessions so future `/121` and `/today` runs start from a richer base.

**If the file doesn't exist**, create it:

```yaml
---
title: "121 — [Person Name]"
description: "[Role/team] — ongoing 1:1 relationship doc with talking points, decisions, and open loops"
type: knowledge
topics: ["leadership"]
created_date: YYYY-MM-DD
---
```

```markdown
# 121 — [Person Name]

## Open Loops
_Unresolved items carried forward across sessions._

## Session Log
```

**On every run**, update the file:

1. **Open Loops section** — refresh: add new open loops from this session's "Close the loop" section, mark resolved ones as `[x]` with the date
2. **Session Log** — prepend a new entry (newest first):

```markdown
### YYYY-MM-DD

**Talking points generated:**
* [bullet from Discuss]
* [bullet from Ask]
* [bullet from Strategic]

**Outcomes:** _Filled in after the meeting by /today or manually._

**New open loops:**
- [ ] [item]
```

The "Outcomes" field starts empty — it gets filled when the user debriefs after the meeting, or when `/today` finds evidence in email/meeting notes the next day.

### Step 6: Confirm

Present the talking points and ask: "Anything to add or adjust?"

## Output Format

- One screen max. Short bullets. No headers beyond the four section names.
- Use `*` bullet style (not `-`) for copy-paste compatibility with most doc tools.
- Sub-bullets with 4-space indent for supporting context under a topic.
- No bold section headers — just the section name followed by a blank line and bullets.
- The user will cherry-pick bullets, so each one should make sense standalone.
