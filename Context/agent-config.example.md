# Agent Config (example)

Copy this file to `Context/agent-config.md` and fill it in — or run `/onboard`, which writes it for you. The real `agent-config.md` is personal and gitignored. All skills read it.

---

## Identity

| Field          | Value                                  |
| -------------- | -------------------------------------- |
| Name           | [Your name]                            |
| Timezone       | [e.g. Australia/Sydney]                |
| Base directory | [Full path to this workspace]          |

---

## Tool Stack

| Role               | Tool            | Value / notes                                              |
| ------------------ | --------------- | ---------------------------------------------------------- |
| Project management | Linear          | Workspace: [name]                                          |
| Email              | Gmail           | Drafts only — never send without explicit approval         |
| Calendar           | Google Calendar | Primary calendar                                           |
| Docs               | Notion          | Draft locally first; never create or edit a Notion page without explicit approval |
| Comms              | none            | No team messaging tool. Skills skip comms steps            |
| Files              | Google Drive    | Read meeting notes (Gemini) and shared docs |
| Meeting notes      | Gemini / pasted | Use pasted notes/transcripts or `Context/Meeting Notes/`   |

---

## Output

All drafts, briefings and summaries show in chat. Files are saved locally where a skill says so. Nothing is sent, posted or published without explicit approval.

---

## Default Roles (for PRDs, experiment docs, goal updates)

| Field                 | Value        |
| --------------------- | ------------ |
| Default driver        | [Your name]  |
| Default approver name | [Name or —]  |

---

## Goals File

Every skill that says `GOALS.md` reads this path:

```
GOALS.md
```

Review-cycle goals file (optional — read by `/achievements` and `/draft-goal-update`):

```
[e.g. Context/Goals H1 2027.md — or none]
```

---

## Calendar Exclusions (meeting-digest, today, plan-week)

Skip events whose title matches any of these:

- Focus Time
- Lunch

Also skip: events with no attendees other than the user, and `focusTime` / `workingLocation` events.
