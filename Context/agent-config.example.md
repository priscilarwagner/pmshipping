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

Every skill reads this table to decide which tool to use. `/onboard` fills it in from your answers. Set a role to `none` and skills skip it. Adapters live in `.claude/adapters/` — copy `_template.md` for a tool that isn't there yet.

| Role       | Tool                          | Adapter                          | Notes                  |
| ---------- | ----------------------------- | -------------------------------- | ---------------------- |
| `pm`       | [e.g. Linear, Jira, none]     | `.claude/adapters/[tool].md`     | [workspace, default team] |
| `email`    | [e.g. Gmail, Outlook, none]   | `.claude/adapters/[tool].md`     |                        |
| `calendar` | [e.g. Google Calendar, Outlook, none] | `.claude/adapters/[tool].md` |                     |
| `docs`     | [e.g. Notion, Confluence, none] | `.claude/adapters/[tool].md`   |                        |
| `files`    | [e.g. Google Drive, none]     | `.claude/adapters/[tool].md`     |                        |
| `comms`    | [e.g. Slack, none]            | `.claude/adapters/[tool].md`     | [priority channels]    |

**Meeting notes sources** (tried in order, first with content wins):
1. [e.g. Calendar event attachments → `files.read`]
2. [e.g. `docs.meeting_notes`]
3. `Context/Meeting Notes/`
4. Ask the user to paste notes

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
