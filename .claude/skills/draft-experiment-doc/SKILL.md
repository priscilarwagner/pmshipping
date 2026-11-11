---
name: draft-experiment-doc
description: "Draft an experiment doc following a standard live experiment doc structure (Background, UX, Development, Post experiment). Confirms which sections are ready for input, pulls context from Linear, PRDs, meeting notes and Notion, then produces a structured doc with filled sections and clear [To complete] markers. Use this skill whenever someone mentions drafting, writing, or creating an experiment doc, experiment brief, or experiment page."
---

## Configuration

Read `Context/agent-config.md` at the start. Use:
- `default_driver` — for the Driver field in the doc header (override if specified)

# Draft Experiment Doc

Draft a structured experiment document following a live doc format. The document has four areas — **Background**, **UX**, **Development**, and **Post experiment** — with different readiness requirements for each. The goal is to create a working doc that's immediately useful: filled sections are complete enough to share, empty sections are clearly marked so nothing gets forgotten.

**Output:** Saved to `Context/Document Hub/[Experiment Name] Experiment Doc.md` and shown in chat for review. Move it to Notion only if the user explicitly asks.

---

## Step 1 — Establish the experiment name

Extract the experiment name from the request. If not provided, ask before continuing — the name anchors everything else.

**Naming convention:** name the experiment by the change being tested (e.g., `Removing the help link from the home page`). If the user's workspace uses a prefix convention, preserve it.

---

## Step 2 — Readiness check

Before gathering context, confirm which sections the user has information ready for. Present this checklist clearly:

```
Which sections do you have info ready for? (I'll pull what I can from Linear, PRDs, meeting notes and Notion — just flag what you have on hand)

🧪 Background
  ☐ Background context — why is this experiment being run?
  ☐ Experiment goal — what outcome are we aiming for?
  ☐ Hypothesis — "We believe that [X] will result in [Y] for [Z users]"

🎨 UX
  ☐ User stories (optional)
  ☐ User flow (optional — Figma or Notion link?)
  ☐ User Testing / Research — any usability work, or reasoning for skipping?

⚙️ Development
  ☐ Experiment setup — variants (control vs treatment), targeting, flag name?
  ☐ Tracking — primary metric, guardrail metrics, events, dashboard?
```

> QA and Post experiment are not needed at doc creation — I'll mark them clearly for later.

For each section the user flags as ready, ask them to paste the content inline. For sections they haven't flagged, you'll still try to pull what's available from context — just note what was inferred vs. what the user provided.

---

## Step 3 — Gather context (run in parallel)

**A. Linear**
Search for a matching issue or project using experiment name keywords:
`list_issues` with `query: "[experiment keywords]"`, and `list_projects` with the same query.
Extract: identifier, description, status, assignee, linked metrics or milestones.

**B. Context/Document Hub/**
Scan for any existing PRD, strategy doc, or prior experiment doc referencing this initiative. If found, reference it in the Background section.

**C. Meeting notes and Notion**
- Scan `Context/Meeting Notes/` from the last 30 days for the experiment keywords
- `notion-search` for related pages and read relevant ones with `notion-fetch`

Extract: decisions made, scope discussions, open questions, metric targets mentioned.

---

## Step 4 — Draft the document

Use the template below. Apply these rules:

- **User-provided content** → fill in directly, no marker needed
- **Inferred from context (Linear/PRD/meeting notes/Notion)** → fill in with a note: *(from Linear)*, *(from meeting notes)* or *(from Notion)*
- **Missing / not provided** → use `[To complete]` — never leave a required section blank without a marker
- **Optional sections with nothing available** → use `[To complete — or mark as not applicable]`

### Document template

```markdown
---
title: [Experiment Name] — Experiment Doc
description: "[~150 chars describing the experiment and its goal — used for future retrieval]"
type: experiment-doc
created_date: [YYYY-MM-DD]
linear_issue: [key or TBD]
status: Planning
---

# [Experiment Name]

| | |
|---|---|
| Driver | [default_driver from agent-config — override if specified] |
| Linear issue | [title + key or TBD] |
| Created | [today's date] |
| Status | Planning |

---

## 🧪 Background

### Background
[Why is this experiment being run? What user problem, business signal, or strategic bet is it responding to?]

### Experiment goal
[What outcome are we trying to achieve? Tie to team or company goals where possible.]

### Hypothesis
*"We believe that [doing X] will result in [outcome Y] for [user segment Z]."*

---

## 🎨 UX

### User stories *(optional)*
[To complete — or: not applicable]

### User flow *(optional)*
[To complete — or: [Figma/Notion link]]

### User Testing / Research
[Link to research, usability testing notes — or: "No user testing planned. Rationale: [___]"]

---

## ⚙️ Development

### Experiment setup

**Variants:**
| Variant | Description |
|---------|-------------|
| Control | [what users see today] |
| Treatment | [what users see in the experiment] |

**Targeting:**
- Audience: [who sees this experiment]
- Rollout: [% or gating logic, e.g. 50/50 split]
- Flag name: [To complete]

### Tracking

**Primary metric:** [metric name + success threshold, e.g. "Support article views — target: -10% (fewer users needing help)"]
**Guardrail metrics:** [what must not regress, e.g. "CSAT, task completion rate"]
**Events to track:** [list key events]
**Dashboard:** [link — or: To complete]

### QA
> ⚠️ *Not required at doc creation — complete before launch.*

[To complete]

---

## 📊 Post experiment

> ⚠️ *Not required at doc creation — fill after the experiment concludes.*

### Experiment results
[To complete]
```

---

## Step 5 — Save the document

Save the completed draft as:
`Context/Document Hub/[Experiment Name] Experiment Doc.md`

---

## Step 6 — Show in chat for review

Display the full doc in chat, then show a compact summary:

```
🧪 **Experiment doc saved — [Experiment Name]**
Saved to: Context/Document Hub/[filename]
Linear issue: [key or TBD]

**Sections filled:** [list sections with real content]
**Needs input before sharing:** [list [To complete] sections that are required]
**Fill later:** QA (before launch), Experiment results (post-experiment)
```

Do NOT create a Notion page unless the user explicitly asks.

---

## Edge cases

- **No Linear issue found:** Draft without it. Note at the top: *"No Linear issue linked — add before sharing."*
- **No UX info at all:** Mark the entire UX section `[To complete — UX section to be filled by design]` and flag it in the chat summary.
- **PRD exists in Context/Document Hub/:** Reference it at the top of the Background section: *"Related PRD: [filename]"*
- **User has everything ready:** Skip the readiness check and draft directly — the checklist is only needed when there's uncertainty about what's available.
