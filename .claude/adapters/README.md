# Tool adapters

Skills never name a tool. They call **role operations** — `pm.my_items(started)`, `email.inbox(since)`, `docs.search(query)` — and the adapter for the tool the user picked turns each operation into real MCP calls.

```
skill  →  role operation  →  Context/agent-config.md (Tool Stack)  →  .claude/adapters/<tool>.md  →  MCP call
```

## How a skill resolves an operation

1. Read `Context/agent-config.md` → **Tool Stack**. Each role names a tool and an adapter file.
2. Open that adapter and follow its recipe for the operation.
3. **Role set to `none`, or the adapter has no recipe for the operation** → skip that step silently and carry on with the rest of the skill. Mention the gap once at the end if it changed the output.
4. **Call fails (auth error, tool not found)** → tell the user which tool is disconnected and point to `/onboard`. Never retry another tool in its place.

Changing tools later = change one row in agent-config. No skill edits.

## Roles and operations

Every adapter implements what its tool can do. Items are whatever the tool calls them (issues, tickets, tasks, cards).

### `pm` — project management

| Operation | Returns |
|---|---|
| `pm.my_items(state)` | Items assigned to the user. `state`: `started`, `todo`, `backlog`, or `open` (all three). Each with title, key, status, priority, due date, project, link |
| `pm.my_items_updated(days)` | The user's items changed in the last N days |
| `pm.my_items_completed(days)` | The user's items completed in the last N days |
| `pm.due(from, to)` | The user's open items with a due date in the range (use `to = yesterday` for overdue) |
| `pm.item(key)` | One item with description |
| `pm.item_comments(key)` | Its comments |
| `pm.search(query)` | Items matching keywords |
| `pm.projects(scope)` | `scope`: `mine` (lead or member) or a name query |
| `pm.project(name)` | One project: description, status, lead, target date |
| `pm.milestones(project)` | Milestones with status and target date |
| `pm.project_items(project, days)` | Items in the project changed in the last N days |
| `pm.project_updates(project)` | Previous status updates, newest first |
| `pm.options(team)` | Teams, projects, labels and statuses available for a new item |
| `pm.create_item(fields)` ✋ | New item: team, title, description, optional project, labels, status, assignee, due date |
| `pm.comment(key, text)` ✋ | Comment on an item |
| `pm.post_update(project, text, health)` ✋ | Project status update |
| `pm.link(item)` | How to write a link to an item |

### `email`

| Operation | Returns |
|---|---|
| `email.inbox(since)` | Real inbox threads since a date — the adapter strips promotions, newsletters, automated mail and calendar notifications |
| `email.sent(since)` | Threads the user sent to since a date |
| `email.search(query, since)` | Threads matching keywords |
| `email.with(person, days)` | Threads to or from a person |
| `email.thread(id)` | Full thread, latest sender first |
| `email.draft(to, subject, body)` ✋ | A draft in the user's mailbox. **Never send** unless the user says "send it" |

### `calendar`

| Operation | Returns |
|---|---|
| `calendar.events(from, to)` | Events with title, time, attendees, description, attachments, event type |
| `calendar.event(id)` | One event |
| `calendar.next_with(person, days)` | Next event with that person as attendee |

### `docs` — long-lived documents

| Operation | Returns |
|---|---|
| `docs.search(query)` | Matching pages with title, author, last edited, link |
| `docs.read(ref)` | Page content |
| `docs.recent_mine(days)` | Pages the user created or edited recently |
| `docs.meeting_notes(date)` | AI meeting notes stored in the docs tool, if it has them |
| `docs.create(title, body)` ✋ | New page |
| `docs.update(ref, section, body)` ✋ | Change one section only |

### `files` — file storage

| Operation | Returns |
|---|---|
| `files.search(query)` | Files with ID, name, type, link |
| `files.read(id)` | File content as text. "Not found" usually means it isn't shared with the user — skip it |

### `comms` — team messaging

| Operation | Returns |
|---|---|
| `comms.mentions(since)` | Messages that mention or DM the user |
| `comms.search(query, since)` | Messages matching keywords |
| `comms.thread(ref)` | Full thread |
| `comms.draft(where, text)` | Text shown in chat for the user to post. **Never post** |

### `meeting_notes` — where meeting content comes from

Not a tool but an ordered list of sources in agent-config, e.g. `calendar attachments → docs.meeting_notes → Context/Meeting Notes/ → ask the user`. Skills try each in order and use the first with content.

✋ = write operation. Show the exact content in chat and wait for an explicit yes before calling it.

## Adding an adapter

Copy `_template.md` to `<tool>.md`, fill in each operation with the tool's real MCP calls (list the server's tools to find them), note the tool's quirks, and mark it `Status: tested` only after running each operation once against a live account.

## Adapters in this repo

| Role | Tested | Untested recipes |
|---|---|---|
| pm | [linear](linear.md) | [jira](jira.md) |
| email | [gmail](gmail.md) | [outlook](outlook.md) |
| calendar | [google-calendar](google-calendar.md) | [outlook](outlook.md) |
| docs | [notion](notion.md) | [confluence](confluence.md) |
| files | [google-drive](google-drive.md) | — |
| comms | — | [slack](slack.md) |
