# Adapter: [Tool name]

**Role:** `[pm | email | calendar | docs | files | comms]` · **MCP:** [server name] · **Status:** untested

Copy this file to `.claude/adapters/<tool>.md`. List the MCP server's tools, fill in a recipe for each operation the tool supports (see `README.md` for the full list), and delete the rows it can't do — skills skip those steps. Mark `Status: tested` only after running each operation once against a live account.

## Quirks

- [Query syntax, date formats, pagination]
- [Noise to filter out]
- [Anything that returned surprising results in testing]

## Operations

| Operation | Recipe |
|---|---|
| `[role].[operation](args)` | `[mcp_tool]` with `[parameters]` |

**Probe (for `/onboard`):** [the cheapest read call that proves the connection works]
