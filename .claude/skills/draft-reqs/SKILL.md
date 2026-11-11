---
name: draft-reqs
description: "Draft a Requirements Doc for a single milestone — the detailed 'what we're building' subpage that lives under the Strategy One Pager. Needs initiative name, milestone name, and ideally context from the Strategy One Pager or opportunity statement."
---

# Draft Requirements Doc

Draft a Requirements Doc for a single milestone following the team template. This is the "what we're building" doc — the level of detail that lets designers design and builders build. One Requirements Doc per milestone.

**Output:** Saved to `Context/Document Hub/[Initiative Name] — [Milestone Name] Requirements Doc.md` + shown in chat for review.

---

## Step 1 — Establish scope

Extract from the request:
- **Initiative name** — which initiative does this belong to?
- **Milestone name** — which milestone is this doc for?

If either is missing, ask before continuing.

---

## Step 2 — Check for upstream context

Look for a Strategy One Pager and/or opportunity statement to inform the doc:

**A. Context/Document Hub/**
Scan for `[Initiative Name] Strategy One Pager.md` or any existing PRD/strategy doc for this initiative.

**B. Linear**
`list_projects` and `list_issues` with `query: "[initiative keywords]"` (limit 10). If a project matches, call `get_project` and `list_milestones`.
Extract: project description, milestone-level issues, assignees.

**C. Notion**
`notion-search` for the Strategy One Pager or related docs; read matches with `notion-fetch`.

If neither a Strategy One Pager nor an opportunity statement is found, note it clearly:

> "No Strategy One Pager or opportunity statement found for [initiative name]. Context will be limited — for a more complete doc, consider running `/draft-idea` and `/draft-prd` first, then come back here."

Proceed anyway if the user wants to continue.

---

## Step 3 — Determine feature type

Ask or infer from context: **Is this an AI-powered feature?**

- If yes → include the Eval cases section (Priority use cases + Golden examples).
- If no → omit the Eval cases section entirely and remove the "Eval cases (AI only)" column from the Requirements table.

If it's not clear from context, ask.

---

## Step 4 — Collect requirements context

Ask the user to provide or confirm:

> "What do you have ready for this milestone? I'll pull what I can from context — flag what you have on hand:
>
> - **Milestone TL;DR** — what does this milestone deliver? why does it sequence here?
> - **Requirements** — list the capabilities or moments to build (I'll structure them into the table)
> - **Design / artifacts** — Figma links or copy tables, if available
> - **Tech design** — doc link or key architectural decisions, if available
> - **Open questions** — anything unresolved

For anything not provided, I'll mark `[To complete]` or pull from the Strategy One Pager / Linear context."

---

## Step 5 — Draft the Requirements Doc

Follow the template structure. Apply these rules:
- **User-provided content** → fill in directly
- **Inferred from Strategy One Pager/Linear/meeting notes** → fill in with note *(from Strategy One Pager)*, *(from Linear)* or *(from meeting notes)*
- **Missing but required** → `[To complete]`
- **Acceptance criteria column** → always `[To be written by engineering]` unless the user provided it
- **AI features only**: include Eval cases section; reference eval case IDs (E1, G1, etc.) in the Requirements table

Structure:

```markdown
---
title: [Initiative Name] — [Milestone Name] Requirements Doc
description: "[~150 chars — what milestone this covers and what it delivers]"
type: requirements-doc
initiative: [initiative name]
milestone: [milestone name]
created_date: [YYYY-MM-DD]
---

# [Initiative Name] — [Milestone Name] Requirements Doc

*Child of: [[Initiative Name] Strategy One Pager]*

## Milestone TL;DR

[Two sentences: what this milestone delivers + acceptance bar, and why it sequences here.]

## Requirements

### [Moment / capability 1]

| # | Requirement | Acceptance criteria | Design / artifact | [Eval cases (AI only)] | Notes / open questions |
|---|-------------|---------------------|-------------------|------------------------|------------------------|
| 1.1 | [outcome statement] | [To be written by engineering] | [link or To complete] | [ref or n/a] | [notes] |

### [Moment / capability 2]

| # | Requirement | Acceptance criteria | Design / artifact | [Eval cases (AI only)] | Notes / open questions |
|---|-------------|---------------------|-------------------|------------------------|------------------------|
| 2.1 | [outcome statement] | [To be written by engineering] | [link or To complete] | [ref or n/a] | [notes] |

[## Eval cases *(for AI features only)*

### Priority use cases (with off-ramps)

| # | Use case | Priority | Off-ramp behaviour |
|---|----------|----------|--------------------|
| E1 | [primary JTBD] | P0 | [fallback] |

### Golden examples & failure modes

| # | Input | Expected behaviour | Pass criteria | Linked requirement |
|---|-------|-------------------|---------------|-------------------|
| G1 | [utterance] | [path + response shape] | [pass condition] | 1.1 |
]

## Tech design

- **Tech design doc:** [link or To complete]
- **Key architectural decisions:** [To complete]
- **Dependencies:** [To complete]

## Open questions

| # | Question | Owner | By when | Status |
|---|----------|-------|---------|--------|
| Q1 | [To complete] | [To complete] | [To complete] | Open |
```

---

## Step 6 — Save

Save as:
`Context/Document Hub/[Initiative Name] — [Milestone Name] Requirements Doc.md`

---

## Step 7 — Show in chat for review

Display the full doc in chat, then show a compact summary:

```
📋 **Requirements Doc saved — [Initiative Name] / [Milestone Name]**
Saved to: Context/Document Hub/[filename]

**Sections filled:** [list sections with real content]
**Needs your input:** [list [To complete] sections — prioritise acceptance criteria + tech design]
**Engineering action:** Acceptance criteria column to be filled by eng before kickoff.

Next: use /draft-experiment-doc to draft the sibling What We Want to Learn doc for this milestone.
```

---

## Edge cases

- **No initiative context at all**: draft a skeleton with all `[To complete]` markers. Note: "Running `/draft-idea` and `/draft-prd` first will give this doc much more to work with."
- **User provides requirements as a list**: convert each bullet into a row in the Requirements table. Group by capability/moment where logical.
- **AI feature but no eval cases yet**: include the Eval cases section structure with all `[To complete]` markers — the section header reminds the team it's needed.
- **Single capability milestone**: use one sub-section only. Don't force multiple sections for a simple milestone.
