---
name: draft-goal-update
description: "Draft a progress update for a goal or project. Gathers context from the Linear project, its issues and milestones, meeting notes and email. Checks for the approver's sign-off. Shows the draft in chat before anything is written to Linear."
---

# Draft Goal Update

Draft the periodic progress update for a goal tracked as a Linear project, following the project's existing update format. Always show in chat for review — never write to Linear until the user confirms.

---

## Configuration

Read at the start:
- `Context/agent-config.md` — for default approver details and the goals file path
- The goals file referenced in agent-config — for the current goal → Linear project map

Do not hardcode goal or project names here.

---

## Instructions

### Step 1 — Resolve the project

If a Linear project name or URL was passed as an argument, use it directly.
If only a goal name was passed, match it to the goals file, then find the project with `list_projects` (query by name).
If neither, ask which goal or project this update is for.

---

### Step 2 — Fetch Linear context (parallel)

Run all of these in parallel:

**A. Project details**
Call `get_project`. Extract:
- Project description / scope and target date
- Target metric and current value (if in description)
- Current status and health

**B. Milestones**
Call `list_milestones` for the project. Note each milestone's status and target date.

**C. Previous updates — extract the template**
Call `get_status_updates` for the project. Find the most recent update. Extract the exact heading format being used and use that structure for the new draft — do not invent a new format.

**D. Issues moved in the last 30 days**
Call `list_issues` with `project: [project]` and `updatedAt: -P30D`. Extract: what was completed, what's in progress, any blockers (blocked status or blocking relations).

---

### Step 3 — Gather supporting context (parallel)

**A. Meeting notes**
Scan `Context/Meeting Notes/` for files from the past 30 days containing the goal or project name. Extract decisions, scope changes and blockers.

**B. Approver sign-off check**
Use `default_approver_name` from `Context/agent-config.md`. Search:
- Meeting notes from the past 30 days that mention the approver and this goal
- Gmail: `search_threads` with `from:[approver] [goal keywords] newer_than:30d`, plus threads between the user and the approver

Classify the sign-off as one of:
- ✅ **Signed off** — explicit approval or agreement on scope/decision. Quote it.
- ⚠️ **Discussed** — relevant conversation found but no explicit sign-off. Quote the closest message.
- ❌ **Not found** — no relevant recent evidence.

If no approver is configured, skip this check and say so in the draft header.

---

### Step 4 — Draft the update and headline sentence

Write the update following the structure from Step 2C. If no previous update exists, use this default structure:

```
Progress Update
[1–2 sentences. What shipped or moved since last update. Lead with the result, not the effort. Be specific — name the milestone or metric change if known.]

What's Changed
[Scope changes, direction shifts, or key decisions made this period. If the approver signed off on something, cite it: "Agreed with [approver] to [decision]." If nothing changed, write "No scope changes."]

What's next
[Specific upcoming milestones. Name them. If dates are known, include them. Not "continuing work" — name the thing.]

Risks & Blockers
[Real blockers only. If none, write "N/A".]
```

Also suggest a project **health** value (On track / At risk / Off track) with a one-line reason.

**Voice rules (from brain.md):**
- Direct, no effort framing — leads with result
- Specific over vague — name things
- Short — no sentence should be padded
- Cite the approver by name when referencing signed-off decisions

**Also draft a headline sentence** — one sentence for a stakeholder-facing summary or deck. Separate from the full update.

Rules for the headline:
- One sentence only — max 20 words
- Written for people who don't track the goal week to week
- Outcome-focused: what changed or what they should know
- A headline, not a summary of the update
- Examples:
  - *"Goal rescoped to focus on activation — personalisation moved to next cycle."*
  - *"Self-serve signup launching next week, unlocking the main driver for this goal."*
  - *"On track — quick-actions milestone shipped to 100% of users."*

---

### Step 5 — Show draft in chat for review

```
📋 **Goal update draft — [Project name]**
[Approver sign-off: ✅ Signed off — "[quote]" OR ⚠️ Discussed — "[quote]" OR ❌ Not found — confirm before posting]
Suggested health: [On track / At risk / Off track] — [reason]

**🎯 Headline**
[One sentence]

**📝 Project update (ready to post):**
[Full update text]
```

Do NOT write to Linear until the user confirms. On confirmation, call `save_status_update` on the project with the update text and health.

---

## Edge cases

- **No previous update found**: use the default structure above and say so in the draft header.
- **Project not found in Linear**: *"⚠️ Couldn't find [project] in Linear. Check the name and try again."*
- **No issues moved this period**: note it in the draft context but do not block — draft from the project description, milestones and meeting notes.
- **Approver sign-off not found**: show the ❌ flag prominently. Do not block the draft.
- **Metric data missing**: write `[metric?]` in the Progress Update section rather than inventing a number.
