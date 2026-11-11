---
name: draft-meeting-minutes
description: "Draft structured meeting minutes from a transcript, notes, calendar event, or verbal description. Saves to Context/Meeting Notes/ and shows in chat for review."
---

# Draft Meeting Minutes

> **Tools:** steps call role operations (`pm.*`, `email.*`, `calendar.*`, `docs.*`, `files.*`, `comms.*`). Resolve each through `Context/agent-config.md` → **Tool Stack** and the adapter it names (`.claude/adapters/README.md`). A role set to `none` → skip that step.

Produce clean, structured meeting minutes from whatever source is available — a transcript, notes, calendar context, or a description of what happened. Save locally and display in chat for review. Nothing is shared unless the user explicitly asks.

---

## Instructions

### Step 1 — Identify the meeting

From the request, extract:
- Meeting name (or infer from context)
- Date (default to today if not specified)
- Attendees (from calendar or request)

Then check for source material in priority order:

**1. Transcript or notes in the request** — if the user pasted a transcript or notes, use them.

**1b. Configured meeting-notes sources** — agent-config → **Meeting notes sources**, in order (e.g. a notes doc attached to the calendar event → `files.read`; `docs.meeting_notes(date)`). Skip any that return nothing or "not found".

**2. Existing meeting note** — scan `Context/Meeting Notes/` for a file matching the meeting name or date.

**3. Calendar event** — `calendar.events` for the meeting date. Extract: title, attendees, description, any attached pre-reads.

**4. Verbal description** — if the user described what happened in their request, use that as the source.

Note which source(s) were used at the top of the draft.

---

### Step 2 — Draft the minutes

```markdown
# [Meeting Name]
*[YYYY-MM-DD] | [Duration if known] | [Attendees — first names or @handles]*
*Source: [transcript / notes / docs / calendar / verbal description]*

## Purpose
[1 sentence — what this meeting was for. Not a list of topics. What was the intent?]

## Decisions
- [Decision — what was locked in, and who made the call if attributable]
- [Each decision is a closed question — something that was open before the meeting and is now resolved]
- *(None made — discussion/update meeting)* if applicable

## Action Items
| # | Task | Owner | Due | Item |
|---|------|-------|-----|-------|
| 1 | [concrete next step] | [Name] | [date or —] | [ticket or —] |

## Key Discussion Points
- [Substantive points raised — the "so what" of each topic, not a transcript summary]
- [Include: options considered, tradeoffs discussed, context shared that affects future decisions]
- [3–7 bullets max. If it doesn't affect a decision or action, leave it out]

## Open Questions & Parking Lot
- [Unresolved questions — what's still open and who owns following up]
- [Topics deferred to a future meeting]

## Context
- [Background or framing that's useful for future reference — max 3 bullets]
- [Links, references, or docs mentioned in the meeting]
```

**Formatting rules:**
- Lead with decisions and actions — that's what people need
- Key discussion points: synthesise, don't transcribe
- Attribute ownership for actions — "[Name] to follow up" not "follow up needed"
- Use [[wiki-links]] for Obsidian: people names, item keys, key concepts

---

### Step 3 — Save to Context/Meeting Notes/

Filename: `YYYY-MM-DD - [Meeting Name].md`. Sanitise the name: replace `/`, `\`, `:` and `|` with ` - `, strip emoji, collapse repeated spaces.

Add YAML frontmatter:
```yaml
---
title: [Meeting Name]
description: "[~150 chars — key decisions or outcomes]"
type: meeting-note
created_date: [YYYY-MM-DD]
---
```

---

### Step 4 — Show in chat for review

Display the full minutes in chat. Then show a compact summary:

```
📋 **Minutes saved** — Context/Meeting Notes/[filename]

**Decisions:** [count or "none"]
**Action items:** [count] — [owners listed]
**Open questions:** [count]

[Action items as bullets if any]

Say "create items" to add action items to your project tool.
```

---

### Step 5 — Create items for action items (if requested)

If the user says "create items" (or "add to [their pm tool]") after reviewing, and the `pm` role is configured:

1. Ask which team (and optionally project) the items belong to, if not obvious (`pm.options`).
2. For each action item without an existing item, check for duplicates with `pm.search`, then `pm.create_item`:
   - Title: the action item text (imperative: "Fix X", "Review Y")
   - Assignee: the owner by name — or the user
   - Description: meeting name, date, 1-sentence context
   - Due date: if one was given

Update the minutes file: fill the Item column with the created keys.

---

## Edge cases

- **No transcript, no calendar, no description**: ask — *"What meeting are these minutes for? Give me the date, attendees, and a quick summary of what happened."*
- **Transcript too long**: summarise by section — don't try to include everything. Signal over noise.
- **Standup or < 5 min meeting**: keep to Purpose + Action Items only. Skip Key Discussion Points.
- **1:1 meeting**: after saving minutes, check `Context/121s/[Person Name].md` and append a session log entry: date, 1-line summary, open loops.
- **Recurring meeting**: check `Context/Meeting Notes/` for previous notes from the same meeting. Note any standing open questions that carry forward.
