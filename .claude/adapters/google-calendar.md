# Adapter: Google Calendar

**Role:** `calendar` · **MCP:** Google Calendar · **Status:** tested

## Quirks

- Pass local times **without** a UTC offset (e.g. `2026-05-18T00:00:00`). Set `timeZone` only when it differs from the user's own.
- Default event types are `DEFAULT`, `OUT_OF_OFFICE`, `FOCUS_TIME`, `FROM_GMAIL`. Treat `FOCUS_TIME` and `WORKING_LOCATION` as blocks, not meetings.
- Meeting notes (e.g. "Notes by Gemini") appear under `attachments` with a Google Docs `fileUrl`. The file ID is the segment after `/d/`. Read it with `files.read`.
- Recurring events come back as individual instances with `recurringEventId`.

## Operations

| Operation | Recipe |
|---|---|
| `calendar.events(from, to)` | `list_events` with `startTime`, `endTime`, `orderBy: startTime` |
| `calendar.event(id)` | `get_event` |
| `calendar.next_with(person, days)` | `list_events` from now to now + `days` with `fullText: {person}`; first match |

**Probe (for `/onboard`):** `list_calendars`.
