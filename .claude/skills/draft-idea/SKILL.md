---
name: draft-idea
description: "Draft an opportunity statement (problem context, impact, desired outcome) and optionally create it as an item in the project-management tool."
---

# Draft Idea

> **Tools:** steps call role operations (`pm.*`, `email.*`, `calendar.*`, `docs.*`, `files.*`, `comms.*`). Resolve each through `Context/agent-config.md` → **Tool Stack** and the adapter it names (`.claude/adapters/README.md`). A role set to `none` → skip that step.

Draft an opportunity statement in the standard product discovery format: problem context, impact, desired outcome. Only use context the user provides — never invent or assume details.

---

## Step 1 — Collect context

Extract from the user's request:
- **Idea title** — what is this about?
- **Problem context** — background or current situation that reveals the problem or unmet need
- **Impact** — how the problem affects customer experience and business objectives
- **Desired outcome** — what success looks like; measurable metrics where possible
- **Resources** *(optional)* — PRD/spec, Loom video, design file, or other links

**Missing required fields:** If any of the three required fields (problem context, impact, desired outcome) are absent or too vague to draft from, ask for them directly before continuing. Batch all missing fields into a single question. Do not proceed until you have enough to write real content — do not use placeholder text for required sections.

Resources are always optional. If the user hasn't mentioned any, do not ask.

---

## Step 2 — Draft the opportunity statement

Use only the information the user has provided. Do not add examples, embellishments, or invented metrics.

```markdown
# 🚀 Opportunity Statement

## Problem context

[User-provided problem context — verbatim or lightly edited for clarity]

## Impact

[User-provided impact — how it affects customer experience and business objectives]

## Desired outcome

[User-provided desired outcome — what success looks like, with metrics if provided]

**Resources:**

[Only include if the user provided links or references. If none, omit this section entirely.]
```

Show the draft in chat and ask: **"Does this look right? Any edits before we move on?"**

Wait for confirmation or edits. Apply changes before continuing.

---

## Step 3 — Ask about the project tool

After the user confirms the draft, ask:

> "Do you want to create this as an item in [pm tool from agent-config]?"

If the `pm` role is `none`, skip Steps 3–7 and offer to save the draft locally instead.

If **no** → done. The draft lives in chat only.

If **yes** → continue to Step 4.

---

## Step 4 — Gather item metadata

Ask the user in a single message:

> "A few details to create it:
> 1. **Team** — which team should own it?
> 2. **Project** *(optional)* — should it sit under an existing project?
> 3. **Labels** *(optional)* — e.g. `Idea`, a theme or area
> 4. **Status** *(optional)* — Triage or Backlog?"

While waiting, run Step 5 in parallel to prepare suggestions.

---

## Step 5 — Look up options

Surface real values so the user doesn't have to remember exact names:
- `pm.options(team)` — teams, projects, labels and statuses

Present them as suggestions alongside the user's answers:

```
Options in [pm tool]:

**Teams:** [list]
**Projects:** [list]
**Labels:** [list]
**Statuses:** [list]
```

If a lookup fails, note it and proceed with whatever the user provides directly.

---

## Step 6 — Create the item

First, check for duplicates with `pm.search(idea title keywords)`. If a matching item exists, show it and ask whether to update it instead.

Once the user confirms the metadata, `pm.create_item` with:

- **Team**: user-confirmed team
- **Title**: the idea title from Step 1
- **Description**: the full opportunity statement from Step 2, in Markdown (headings: `Problem context`, `Impact`, `Desired outcome`, and `Resources` if provided)
- **Project / Labels / Status**: if provided

---

## Step 7 — Confirm and link

After creation, show:

```
✅ Idea created: [Title] ([KEY-123](link))

Fields set:
- Team: [team]
- Project: [project or —]
- Labels: [labels or —]
- Status: [status]
```

If any field couldn't be set, note it:
> "[Field] couldn't be set automatically — update it manually at [link]."

---

## Edge cases

- **Not enough context for a required field**: ask the user rather than drafting a placeholder. One gap = one short follow-up question.
- **Auth error**: tell the user which tool is failing and direct them to reconnect it.
- **Team or project not found in Step 5**: ask the user to confirm the name, then retry once.
- **User wants to save the draft locally**: save to `Context/Document Hub/[Title] — Idea.md` only if the user asks. Do not save automatically.
