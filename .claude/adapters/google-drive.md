# Adapter: Google Drive

**Role:** `files` · **MCP:** Google Drive · **Status:** tested

## Quirks

- `read_file_content` needs an exact file ID. From a Docs URL, take the segment after `/d/`.
- **"Requested entity was not found"** usually means the file isn't shared with the user (e.g. meeting notes owned by another organiser). Skip it — don't retry.
- Read-only in this harness: no writes, shares or trashing.

## Operations

| Operation | Recipe |
|---|---|
| `files.search(query)` | `search_files` |
| `files.read(id)` | `read_file_content` with `fileId` |

**Probe (for `/onboard`):** `list_recent_files`, limit 1.
