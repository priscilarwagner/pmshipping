# Adapter: Notion

**Role:** `docs` · **MCP:** Notion · **Status:** tested

## Quirks

- `notion-search` needs a non-empty `query` unless you pass filters. Filters like `edited_by_user_ids` and `last_edited_date_range` need a Business plan; on other plans they're dropped with a notice.
- `notion-query-meeting-notes` needs Notion AI meeting notes enabled. An empty result is normal when the user doesn't record in Notion.
- Read full pages with `notion-fetch` (URL or ID).

## Operations

| Operation | Recipe |
|---|---|
| `docs.search(query)` | `notion-search` with `query`, `page_size: 10` |
| `docs.read(ref)` | `notion-fetch` |
| `docs.recent_mine(days)` | `notion-search` with `query: ""`, `filters: {edited_by_user_ids: ["me"]}`, `sort: last_edited`; keep pages edited in the window. On non-Business plans, fall back to a keyword search |
| `docs.meeting_notes(date)` | `notion-query-meeting-notes` with a `created_time` filter for that date |
| `docs.create(title, body)` ✋ | `notion-create-pages` |
| `docs.update(ref, section, body)` ✋ | `notion-update-page`, changing only that section |

**Probe (for `/onboard`):** `notion-search` with a simple query.
