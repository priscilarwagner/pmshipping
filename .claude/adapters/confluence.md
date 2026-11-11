# Adapter: Confluence

**Role:** `docs` · **MCP:** Atlassian · **Status:** untested — verify tool names against your Atlassian MCP before relying on it

## Quirks

- Search uses CQL (`text ~ "..."`, `contributor = currentUser()`).
- Page bodies are in storage format (XHTML). When updating, change only the target section and keep the rest byte-for-byte.
- Confluence has no AI meeting-notes store; leave `docs.meeting_notes` out of the `meeting_notes` sources.

## Operations

| Operation | Recipe |
|---|---|
| `docs.search(query)` | `searchConfluenceUsingCql` with `text ~ "{query}"` |
| `docs.read(ref)` | `getConfluencePage` |
| `docs.recent_mine(days)` | CQL `contributor = currentUser() AND lastmodified >= now("-{days}d")` |
| `docs.meeting_notes(date)` | Not supported |
| `docs.create(title, body)` ✋ | `createConfluencePage` |
| `docs.update(ref, section, body)` ✋ | `getConfluencePage`, edit one section, `updateConfluencePage` |

**Probe (for `/onboard`):** CQL search with a simple keyword, limit 1.
