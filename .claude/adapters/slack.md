# Adapter: Slack

**Role:** `comms` · **MCP:** Slack · **Status:** untested — verify tool names against your Slack MCP before relying on it

## Quirks

- Search syntax: `to:me`, `from:@name`, `in:#channel`, `after:YYYY-MM-DD`.
- Skip bot messages and channel joins unless they name the user.
- A thread is actioned when the user has replied or reacted in it.
- **Never post.** Drafts are shown in chat; the user posts them.

## Operations

| Operation | Recipe |
|---|---|
| `comms.mentions(since)` | `slack_search_public_and_private` with `to:me after:{since}` |
| `comms.search(query, since)` | `slack_search_public_and_private` with `{query} after:{since}` |
| `comms.thread(ref)` | `slack_read_thread` |
| `comms.draft(where, text)` | Show in chat only |

Priority channels for briefings go in agent-config under **Comms channels**.

**Probe (for `/onboard`):** `slack_search_public_and_private` with `to:me`, limit 1.
