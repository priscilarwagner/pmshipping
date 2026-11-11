---
name: morning-briefing
description: "Daily brief — pulls calendar, Linear and overnight email signals, picks 3 focus items, and saves a compact briefing to Notes/Briefings/. Can run unattended on a weekday schedule."
---

## Configuration

Read `Context/agent-config.md` at the start. Use:
- `timezone` — for date calculations throughout
- `base_directory` — for resolving skill and output file paths
- **Tool Stack** — which tools to pull from (Linear, Gmail, Google Calendar; no comms tool)

# Morning Briefing

Pull today's signals from all connected tools, synthesise into a compact brief, and save it locally. No interaction required — this can run unattended.

This is NOT `/today`. It does not do deep 1:1 prep, does not update the weekly scratchpad, and does not ask for confirmation. It is a read-only morning snapshot.

---

## Instructions

Run all steps in sequence. If a tool is unavailable or returns an error, skip that section gracefully — write what's available and note what was skipped.

---

### Step 1 — Get today's context

Compute today's date and yesterday's date using the timezone from `Context/agent-config.md`.

**Calendar** — call Google Calendar `list_events` for today (midnight → 11:59pm local time):
- Extract: meeting title, time, duration, attendees
- Flag: 1:1s (2 attendees or title contains "1:1", "121", "catch up", person name only)
- Flag: back-to-back stretches (< 15 min between meetings)
- Flag: any deep work blocks or focus time events

**Linear** — `list_issues` has no due-date filter, so fetch and filter locally:
1. `list_issues` with `assignee: "me"`, `state: "started"` (In Progress + In Review), `fields: [id, title, status, priority, dueDate, project, url, updatedAt]`
2. `list_issues` with `assignee: "me"`, `state: "unstarted"` and again with `state: "backlog"`, same fields — keep only issues with a `dueDate`
3. From all results: **overdue** = `dueDate` before today; **due today** = `dueDate` = today

Deduplicate by issue identifier. Max 8 issues total.

**Email** — call Gmail `search_threads` for overnight mail (since 6pm yesterday):
```
in:inbox after:[YYYY/MM/DD yesterday] -from:me -category:promotions -category:updates -category:social -category:forums
```
Use `view: THREAD_VIEW_SUMMARY`. Extract: questions directed at the user, decisions or deadlines (interviews, recruiters, offers, contracts), anything blocking someone. Ignore newsletters, receipts and automated notifications (`no-reply`, `noreply`, `notifications@`) unless they carry a deadline.

**Draft request detection** — while scanning email, flag any thread that asks the user for something matching these patterns:

| Pattern | Draft type | Example trigger |
|---------|-----------|-----------------|
| "update", "progress", "goal", "status", "deadline", "by [day]" + project/goal context | `goal_update` | A stakeholder asking for a project status update |
| "PRD", "product brief", "write up", "kickoff doc", "requirements" | `prd` | Someone asking for a PRD on a new initiative |
| "notes", "minutes", "recap", "write up the meeting", "can you document" | `meeting_minutes` | Someone asking for notes after a sync |

For each match, extract:
- What specifically is being requested
- The relevant issue, project, meeting, or initiative name
- The deadline, if mentioned
- Who asked

Store all matches as `DRAFT_TRIGGERS` — used in Step 1b below.

**Tasks** — scan `Tasks/` for files with `status: s` (in progress) or `priority: P0`. Note titles and due dates only — do not read full files.

---

### Step 1b — Auto-draft (if triggered)

If `DRAFT_TRIGGERS` is non-empty, run this step before writing the morning brief.

For each triggered item, call the `/draft` router:

1. Read `[base_directory]/.claude/skills/draft/SKILL.md` and execute it, passing the full context of what was requested.
2. The `/draft` skill saves or shows its output locally — nothing is sent.
3. Add a line to the morning brief under **⚠️ Heads Up**:
   > *🖊️ Draft ready: [type] — [subject]. See [file path].*

If `DRAFT_TRIGGERS` is empty, skip this step entirely.

---

### Step 2 — Pick 3 focus items

Select exactly 3 focus items for today. Priority order:
1. Overdue Linear issues (due date passed, not done)
2. P0 tasks from `Tasks/`
3. Linear issues due today
4. In-progress Linear issues most recently updated
5. Active tasks (`status: s`) by priority field

State each item as: what it is, why it's the priority today (due date, overdue, or goal ref).

---

### Step 3 — Compose the brief

Markdown. Keep the whole brief under 2800 characters. Omit any section where there's nothing to report — don't write "None" or empty headers.

```
## ☀️ Morning Brief — [Weekday, DD Mon]

**📅 Today's Calendar**
- [HH:MMam] — [Meeting name] ([duration]) [🔁 if recurring] [👥 1:1 flag if applicable]
- ...
_[Flag if back-to-back stretches exist, e.g. "⚡ Heads up: 10am–1pm is back-to-back"]_

**🎯 Top 3 for Today**
1. **[Item]** — [why: overdue / due today / goal ref]
2. **[Item]** — [why]
3. **[Item]** — [why]

**🔴 Linear — Overdue**
- [Issue title] ([KEY-123](issue-url)) _(due [date])_

**🟡 Linear — Due Today**
- [Issue title] ([KEY-456](issue-url))

**✉️ Overnight Email**
- [1-line summary of anything actionable or decision-relevant. No FYIs.]

**⚠️ Heads Up**
- [Any risk, blocker, or time-sensitive thing worth knowing before starting the day]
```

Rules:
- Name issues by title first, then link the key, using the `url` returned by Linear
- Omit the *Overnight Email* section if nothing actionable was found
- Omit the *Heads Up* section if nothing to flag
- Omit *Linear — Overdue* or *Linear — Due Today* sections individually if empty
- If calendar is empty, write: `- No meetings today 🎉`
- Do not add "have a great day" or any motivational language

---

### Step 4 — Save and show

Write the brief to `Notes/Briefings/YYYY-MM-DD.md` (create the folder if missing) and print it in chat.

When running unattended, output nothing else except a single confirmation line for the log: `morning-briefing saved`.

---

## Edge cases

- **Weekend run**: If today is Saturday or Sunday, write a single line: `🗓️ No brief today — it's [Saturday/Sunday].` and exit.
- **Public holiday / OOO**: If the calendar shows an all-day OOO or public holiday event, note it at the top of the brief: `_📴 OOO today — brief is for reference only._`
- **All tools fail**: Write: `## ☀️ Morning Brief — [date]` and `⚠️ Could not reach tools (Calendar, Linear, Gmail). Check connections.`
- **No Linear items**: Omit both Linear sections entirely.
- **No overnight email signals**: Omit the Overnight Email section.
