---
name: meeting-digest
description: "Digest all meetings from a given day — uses Google Calendar as source of truth and the user's notes or transcripts as content. Extracts decisions, action items, and blockers, writes structured notes to Context/Meeting Notes/."
---

# Meeting Digest

Fetch all meetings for a target day, find the notes or transcript for each, produce a structured digest, and save it locally. The target day defaults to today; pass "yesterday" to process the prior day.

## Configuration

Read `Context/agent-config.md` at the start. Use:
- `timezone` — for all date calculations
- `base_directory` — for file path construction and idempotency check
- `calendar_exclusions` — list of meeting title patterns to skip
- **Tool Stack** → Meeting notes — where meeting content comes from

Read the goals file (path in agent-config) to recognise project and milestone names in Step 4.

---

## Instructions

Run all steps in sequence. Do not ask for confirmation between steps unless explicitly told to pause.

---

### Step 1 — Fetch all undigested meetings for the target day

#### Phase A — Calendar: get the meeting list

1. Compute the target date in the user's timezone. Default = today; if the user passed "yesterday", subtract one day.

2. Call Google Calendar `list_events` with:
   - `startTime` = start of target date in local timezone (ISO 8601 with offset, e.g. `2026-05-18T00:00:00+10:00`)
   - `endTime` = end of target date in local timezone (e.g. `2026-05-18T23:59:59+10:00`)
   - `timeZone` = user's timezone
   - `orderBy` = `startTime`

3. **Filter to real meetings only.** Remove any event where:
   - `summary` matches a pattern in `calendar_exclusions`
   - `eventType` is `focusTime` or `workingLocation`
   - `attendees` list is empty or contains only the user (solo blocks)

4. **Sort chronologically** (oldest first). This is the ordered list of meetings to process.

#### Phase B — Idempotency check

For each meeting:

1. Derive the expected notes filename: `YYYY-MM-DD - [Meeting Name].md` using the calendar event `summary`. Sanitise the name: replace `/`, `\`, `:` and `|` with ` - `, strip emoji, collapse repeated spaces.
2. Check if a file with that name already exists in `[base_directory]/Context/Meeting Notes/`.
3. If it exists and already has a `## Summary` section → skip this meeting. Output: *"Notes already exist for [meeting name] — skipping."*
4. If it exists as raw notes (no `## Summary`) → use its content as the source and proceed.
5. If it does not exist → proceed to Phase C.

#### Phase C — Find the meeting content

Check sources in this order and use the first one that has content:

1. **Raw notes file** — from Phase B step 4.
1b. **Gemini notes** — if the calendar event has an attachment titled "Notes by Gemini" (or any Google Doc attachment with notes), read it with Google Drive `read_file_content` using the file ID from the attachment URL. If it returns "not found", the organiser hasn't shared it — skip silently.
2. **Notion meeting note** — call `notion-query-meeting-notes` for the target date and match on title or time window (±30 min). Read it with `notion-fetch`.
3. **Calendar event description** — use only if it contains real notes, not just an agenda or a video link.
4. **Ask the user** — list the meetings that still have no content in one message: *"No notes found for: [list]. Paste notes or a transcript for any you want digested, or say skip."*

If a meeting still has no content → mark as `"no notes available"` and skip it.

#### Phase D — Report

After processing all meetings, report:
- How many were digested
- How many were skipped (with reason: already exists / no notes)

---

### Step 2 — Read context

Before processing any note content, read:
- `GOALS.md` — to understand active projects and goals
- `Context/Memory/brain.md` — to understand how the user thinks, their vocabulary, and what they consider a decision vs. a discussion

---

### Step 3 — Analyse the note content

For each meeting with content, extract four things:

**A. Summary**
2–3 sentences. Signal-first: what was the meeting about and what was the key outcome or shift. Not a list of topics covered — a single coherent outcome statement. Use the user's voice: direct, no effort framing, leads with result.

**B. Decisions**
Explicit commitments that changed direction, locked something in, or closed an open question. Only real decisions — not suggestions, not maybes. Format: what was decided, and who made the call (if attributable).

**C. Action items**
Concrete next steps with a named owner. Apply this logic per item:
- If the note names a specific person as the driver with high confidence → assign to them
- If ownership is ambiguous or shared → assign to the user
- Only include a Linear issue or project if it was **explicitly mentioned** in the meeting (by key, name, or clear reference). Do not infer or guess — if none was mentioned, leave the Project column blank (`—`)
- Note the suggested due date if mentioned in the meeting, otherwise leave blank

**D. Open questions / blockers**
Unresolved questions, blockers raised, or topics parked for a future meeting. These go in the notes file only.

---

### Step 4 — Project references

Only record a Linear issue key or project name when it was **explicitly mentioned** in the meeting — by key (e.g. ENG-123), by name (e.g. "the onboarding revamp milestone"), or by unambiguous reference.

Do not infer, look up, or guess projects from meeting context. It is better to leave it blank than to assign a wrong one.

---

### Step 5 — Write the meeting notes file

Save the digest to `Context/Meeting Notes/` using this filename format:
`YYYY-MM-DD - [Meeting Name].md`

Use the calendar event `summary` as the meeting name.

**File format:**

```markdown
# [Meeting Name]
*[YYYY-MM-DD] | [Duration if available] | [Attendees from calendar if available]*

## Summary
[2–3 sentence signal-first summary. What happened and what changed.]

## Decisions
- [Decision 1 — who made it if known]
- [Decision 2]

## Action Items
| # | Task | Owner | Project | Due |
|---|------|-------|---------|-----|
| 1 | [Task description] | [Name / me] | [Project name (KEY-123) or —] | [Date or —] |
| 2 | ... | ... | ... | ... |

## Open Questions & Blockers
- [Question or blocker — what's unresolved and why it matters]

## Context
[Any background, links, or framing from the meeting that's useful for future reference. Keep to 3–5 bullets max.]
```

If raw notes were the source (Phase B step 4), keep them under a `## Raw notes` section at the bottom — never delete the user's notes.

**Before saving**, add `[[wiki-links]]` to the note for Obsidian graph connectivity:
- All people mentioned by name → `[[Name]]`
- All Linear issue keys → `[[KEY-123]]`
- Key concepts, projects and themes that recur across notes → `[[Concept]]`
- Only link meaningful mentions — not every word, not inside code blocks or URLs

Confirm to the user: *"Meeting notes saved to Context/Meeting Notes/[filename].md"*

---

### Step 6 — Compound on touch

After saving:
1. Check if any decision or blocker from this meeting relates to an open task in `Tasks/` — if so, note it inline.
2. If a blocker was raised that already exists in `GOALS.md` as a known blocker, flag it: *"This blocker ([name]) was also raised in [previous context]. Consider whether it's systemic."*
3. If the meeting is a recurring 1:1 with a named person who has a doc in `Context/121s/`, append a session log entry to their doc: date, 1-line summary, any open loops added.

Meeting notes contain personal data and are never committed or pushed.

---

### Step 7 — Show summary in chat

After saving all notes locally, display one combined summary for the full day's digest.

Structure: meeting → topic → decisions + actions under each topic.

```
📝 **Meeting digest complete — [YYYY-MM-DD]** | [N] digested | [N] skipped

*[Meeting Name]*
  [Topic / goal or milestone]
    Decisions
      - [Decision]
      - [Decision]
    Action items
      - [task] → [owner] [due if known]

  [Topic 2]
    Decisions
      - [Decision]
    Action items
      - [task] → [owner]

Notes saved to `Context/Meeting Notes/`
```

**Grouping rules:**
- Top level: meeting name
- Second level: topic, goal, or milestone discussed within that meeting — use the plain name, never a bare issue key
- Third level: **Decisions** and **Action items** as labelled sub-sections under each topic
- If a topic has no decisions, omit the Decisions sub-section. If it has no action items, omit the Action items sub-section.
- If a meeting has only one topic, still show the topic name — don't collapse it
- Omit the Project column from action items in the chat summary — it belongs in the notes file only

---

### Step 8 — Offer decision log update

This step only runs if at least one decision was extracted across all digested meetings.

Ask once, in a single message:

> "Want to log any of these decisions to a PRD's decision log?
>
> **Decisions from today:**
>
> *[Meeting Name]*
>   [Topic]
>     1. [Decision]
>     2. [Decision]
>
> If yes, tell me which doc (a local file in `Context/Document Hub/` or a Notion page URL) and which decision numbers to add."

Number decisions sequentially across all meetings so the user can reference them by number.

Wait for the user's response.

- **If no / skip**: end the skill.
- **If yes**: proceed with the steps below for each doc they specify.

---

#### For each doc provided:

**A. Fetch the doc**

- Local file → read it.
- Notion page → call `notion-fetch` with the page URL or ID.

**B. Locate the Decision Log**

Scan for a section whose heading contains "Decision Log", "Decisions Log", or "Decisions" (case-insensitive).

- **If a Decision Log section with a table exists**: new rows will be appended to it.
- **If a Decision Log section exists but has no table**: insert the table immediately under the heading.
- **If no Decision Log section exists**: insert a new `## Decision Log` section and table at the bottom of the doc.

**C. Build the rows**

For each decision the user selected, create one table row:

```
| [YYYY-MM-DD] | [Decision text] | [Status] |
```

- **Date**: the meeting date.
- **Decision**: exact text from the digest — do not paraphrase.
- **Status**: `Decided` if explicit and final. `Pending` if the text contains "TBD", "to be confirmed", "pending", "waiting on", or was marked unresolved.

If the table doesn't exist yet, create it with this header row first:

```markdown
| Date | Decision | Notes/Status |
|------|----------|--------------|
```

**D. Preview, then update**

Show the rows to be added. For a local file, write them. For a Notion page, wait for the user's explicit OK, then call `notion-update-page` — change only the Decision Log section.

**E. Confirm**

> "Decision Log updated on [doc title] — added [N] decision(s)."

---

## Edge cases

- **No decisions found**: Write "No decisions made — discussion/update meeting." Do not invent decisions.
- **No action items found**: Write "No action items."
- **Meeting is a standup or <5 min**: Still run the digest, but keep the summary to 1 sentence.
- **Multiple Notion notes match the same event**: Use the one closest to the event's start time.
- **Notion unavailable**: Skip that source silently and fall through to the next one in Phase C.
- **Notion page not found (Step 8)**: Report the error, ask the user to double-check the URL, and offer to retry.
- **Decision Log section ambiguous (Step 8)**: If multiple headings could match (e.g. "Key Decisions" and "Decision Log"), show both and ask which one to append to.
- **Duplicate decisions (Step 8)**: Before inserting, scan existing rows. If the same decision text already appears, skip it and note the skip.
