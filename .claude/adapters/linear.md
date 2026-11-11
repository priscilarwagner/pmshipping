# Adapter: Linear

**Role:** `pm` · **MCP:** Linear · **Status:** tested

## Quirks

- `list_issues` has **no due-date filter**. For anything date-based, fetch open items and filter on `dueDate` yourself.
- `state` accepts a state *type*: `started` (In Progress + In Review), `unstarted` (Todo), `backlog`, `completed`, `canceled`. Status *names* differ per team — `list_issue_statuses` lists them.
- Durations are ISO-8601: `-P7D` = last 7 days.
- Always pass `fields` so results include `dueDate`, `priority`, `project` and `url`.
- Results paginate: when `hasNextPage` is true, pass `cursor`.

## Operations

| Operation | Recipe |
|---|---|
| `pm.my_items(state)` | `list_issues` with `assignee: "me"`, `state: started \| unstarted \| backlog`, `fields: [id, title, status, priority, dueDate, project, url, updatedAt]`. For `open`, run all three |
| `pm.my_items_updated(days)` | `list_issues` with `assignee: "me"`, `updatedAt: -P{days}D` |
| `pm.my_items_completed(days)` | `list_issues` with `assignee: "me"`, `completedAt: -P{days}D` |
| `pm.due(from, to)` | `pm.my_items(open)`, keep items whose `dueDate` falls in the range |
| `pm.item(key)` | `get_issue` with the identifier (e.g. `ENG-123`) |
| `pm.item_comments(key)` | `list_comments` with the issue ID |
| `pm.search(query)` | `list_issues` with `query` (and `team` if known) |
| `pm.projects(scope)` | `mine` → `list_projects` with `member: "me"`; name → `list_projects` with `query` |
| `pm.project(name)` | `get_project` |
| `pm.milestones(project)` | `list_milestones` with `project` |
| `pm.project_items(project, days)` | `list_issues` with `project`, `updatedAt: -P{days}D` |
| `pm.project_updates(project)` | `get_status_updates` with `type: "project"`, `project` |
| `pm.options(team)` | `list_teams`, `list_projects` (team), `list_issue_labels` (team), `list_issue_statuses` (team) |
| `pm.create_item(fields)` ✋ | Check duplicates with `pm.search` first, then `save_issue` (omit `id`) with `team`, `title`, `description` (Markdown), optional `project`, `labels`, `state`, `assignee`, `dueDate` |
| `pm.comment(key, text)` ✋ | `save_comment` |
| `pm.post_update(project, text, health)` ✋ | `save_status_update` with `type: "project"`, `project`, `body`, `health` (`onTrack` / `atRisk` / `offTrack`) |
| `pm.link(item)` | Use the `url` field: `[Title](url) (KEY-123)` |

**Probe (for `/onboard`):** `list_issues` with `assignee: "me"`, `limit: 1`.
