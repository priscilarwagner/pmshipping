# Adapter: Outlook (Microsoft 365)

**Roles:** `email`, `calendar` · **MCP:** Microsoft 365 · **Status:** untested — Microsoft 365 MCPs differ; list your server's tools during `/onboard` and fill in the exact names below

## Quirks

- Mail search usually takes KQL (`from:`, `received>=`) rather than Gmail operators.
- Use the **Focused** inbox (or exclude the Other folder) as the noise filter.
- Skip meeting request / response messages — the calendar already holds them.
- Teams meeting transcripts and recaps may be reachable through the same server; if so, list them as a `meeting_notes` source in agent-config.

## Operations

| Operation | Recipe |
|---|---|
| `email.inbox(since)` | Search mail in Inbox (Focused), `received>={since}`, not from the user |
| `email.sent(since)` | Search Sent Items, `sent>={since}` |
| `email.search(query, since)` | Search mail with `{query}` and `received>={since}` |
| `email.with(person, days)` | Search mail `from:{person} OR to:{person}` in the window |
| `email.thread(id)` | Get the conversation by ID |
| `email.draft(to, subject, body)` ✋ | Create a draft message. Never send |
| `calendar.events(from, to)` | List calendar view between the two times |
| `calendar.event(id)` | Get event |
| `calendar.next_with(person, days)` | Calendar view for the window, filter by attendee |

**Probe (for `/onboard`):** list one inbox message and one calendar event.
