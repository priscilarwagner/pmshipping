# Adapter: Gmail

**Role:** `email` · **MCP:** Gmail · **Status:** tested

## Quirks

- **Noise filter** — append to every inbox query: `-category:promotions -category:updates -category:social -category:forums`.
- **Don't use `category:primary`.** It can return nothing when mail is labelled `CATEGORY_PERSONAL`.
- **Skip calendar notifications** — subjects starting `Invitation:`, `Updated invitation`, `Accepted:`, `Declined:`, `Appointment cancelled`, and booking confirmations. The calendar already holds them.
- **Skip automated senders** — `no-reply`, `noreply`, `notifications@` — unless the mail carries a deadline.
- `view: THREAD_VIEW_SUMMARY` returns only the latest message per thread: if its sender is the user, the thread is already actioned.
- `after:` / `before:` take `YYYY/MM/DD` and evaluate at midnight.
- `get_thread` with `messageFormat: PLAIN_TEXT` keeps threads readable.

## Operations

| Operation | Recipe |
|---|---|
| `email.inbox(since)` | `search_threads` with `in:inbox after:{since} -from:me` + noise filter, `view: THREAD_VIEW_SUMMARY`; drop calendar notifications |
| `email.sent(since)` | `search_threads` with `in:sent after:{since}` |
| `email.search(query, since)` | `search_threads` with `{query} after:{since}` |
| `email.with(person, days)` | `search_threads` with `(from:{person} OR to:{person}) newer_than:{days}d` |
| `email.thread(id)` | `get_thread` with `messageFormat: PLAIN_TEXT` |
| `email.draft(to, subject, body)` ✋ | `create_draft`. Never call a send tool |

**Probe (for `/onboard`):** `search_threads` with `in:inbox`, `pageSize: 1`.
