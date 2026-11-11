# Adapter: Jira

**Role:** `pm` · **MCP:** Atlassian · **Status:** untested — verify tool names against your Atlassian MCP before relying on it

## Quirks

- Everything goes through JQL. `currentUser()` is the signed-in user.
- Some calls need a `cloudId` — get it once from `getAccessibleAtlassianResources` and store it in agent-config notes.
- Due dates *can* be filtered in JQL (`due <= now()`).
- Descriptions and comments may need Atlassian Document Format; check the MCP's create/edit parameters.

## Operations

| Operation | Recipe |
|---|---|
| `pm.my_items(state)` | `searchJiraIssuesUsingJql`: `assignee = currentUser() AND statusCategory = "In Progress"` (started), `= "To Do"` (todo/backlog), `!= Done` (open) |
| `pm.my_items_updated(days)` | JQL `assignee = currentUser() AND updated >= -{days}d` |
| `pm.my_items_completed(days)` | JQL `assignee = currentUser() AND statusCategory = Done AND resolved >= -{days}d` |
| `pm.due(from, to)` | JQL `assignee = currentUser() AND statusCategory != Done AND due >= "{from}" AND due <= "{to}"` |
| `pm.item(key)` | `getJiraIssue` |
| `pm.item_comments(key)` | `getJiraIssue` with comments expanded |
| `pm.search(query)` | JQL `text ~ "{query}" ORDER BY updated DESC` |
| `pm.projects(scope)` | `getVisibleJiraProjects` (or epics: JQL `issuetype = Epic AND assignee = currentUser()`) |
| `pm.project(name)` | `getJiraIssue` on the epic, or project details |
| `pm.milestones(project)` | Child epics or fix versions of the project |
| `pm.project_items(project, days)` | JQL `parent = {epic} AND updated >= -{days}d` |
| `pm.project_updates(project)` | Comments on the epic that follow an update format |
| `pm.options(team)` | `getVisibleJiraProjects`, then issue types and fields for the chosen project |
| `pm.create_item(fields)` ✋ | `createJiraIssue`; custom fields may need `editJiraIssue` afterwards |
| `pm.comment(key, text)` ✋ | `addCommentToJiraIssue` |
| `pm.post_update(project, text, health)` ✋ | `addCommentToJiraIssue` on the epic |
| `pm.link(item)` | `https://{site}.atlassian.net/browse/{KEY}` |

**Probe (for `/onboard`):** JQL `assignee = currentUser() ORDER BY updated DESC`, `maxResults: 1`.
