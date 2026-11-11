---
name: draft-prd
description: "Draft a PRD using an AI-era PRD template (Opportunity Space + Solution Space). Pulls context from the project-management tool, GOALS.md, meeting notes, the docs tool and existing local docs. Saves to Context/Document Hub/ and shows in chat for review."
---

# Draft PRD

> **Tools:** steps call role operations (`pm.*`, `email.*`, `calendar.*`, `docs.*`, `files.*`, `comms.*`). Resolve each through `Context/agent-config.md` → **Tool Stack** and the adapter it names (`.claude/adapters/README.md`). A role set to `none` → skip that step.

Draft a product requirements document following the AI-era PRD template below (Opportunity Space → Solution Space). Populate every required section from available context — leave optional sections clearly marked as [To complete].

**Output:** Saved to `Context/Document Hub/[Initiative Name] PRD.md` + shown in chat for review. Publish it to the docs tool (`docs.create`) only if the user explicitly asks.

## Configuration

Read `Context/agent-config.md` at the start. Use:
- `default_driver` — for the Driver field in the PRD header (override if a different driver is specified)
- `default_approver_name` — for the Approver field (override if specified)

---

## Instructions

### Step 1 — Establish scope

Extract from the request:
- **Initiative name** — what is being built?
- **Project** — is there an existing project or item in the project-management tool? (name, URL, or key)
- **Stage** — pre-kickoff (Opportunity Space only) or post-kickoff (full doc)?

If initiative name is missing, ask before proceeding.

---

### Step 2 — Gather context (parallel)

**A. Project management**
If a project is provided:
- `pm.project(name)` — description, status, lead, target date
- `pm.milestones(project)` — milestones
- `pm.project_items(project, 90)` — items for scope and key features

If no project:
- `pm.projects(keywords)` and `pm.search(keywords)` — find anything related

**B. GOALS.md**
Read `GOALS.md` — identify which goal this initiative serves. Extract success metrics and strategic context.

**C. Context/Document Hub/**
Scan for any existing PRD, strategy doc, or decision record referencing this initiative.

**D. Meeting notes and docs**
- Scan `Context/Meeting Notes/` from the last 30 days for the initiative name
- `docs.search(keywords)` for related pages (strategy docs, research, prior PRDs); read the relevant ones with `docs.read`

Extract: decisions made, scope discussions, open questions.

---

### Step 3 — Draft the PRD

Follow the template structure exactly. Mark required sections clearly.

---

#### Header

```markdown
# [Cycle] - [Initiative Name]

| | |
|---|---|
| Driver | [default_driver from agent-config — override if specified] |
| Approver | [default_approver_name from agent-config — override if specified] |
| Contributors | [from project members / meeting context] |
| Informed | [from meeting / email context] |
| Project | [name + link or TBD] |
| Status | [Kicking off / Shaping / Building / Rollout] |

### Change log
| Date | Change |
|------|--------|
| [today] | Initial draft |
```

---

#### 1. Opportunity Space

**🚀 Problem**
[Why this matters for the business. Tie to company strategy. Pull from the project description or GOALS.md.]

**Who: Target Users** ✅
[Who are you building for? Be specific — logged-out users, Pro users, Enterprise admins, etc.]

**Problem Statement** ✅
- Customer problem: [what friction or gap exists for users]
- Business problem: [what it costs the business to leave this unsolved]

**Why & Why Now: Strategic fit** ✅
- Strategic fit: [which company or team goal does this serve]
- Why now: [what's changed — new capability, data signal, competitive move]
- Non-goals: [what this explicitly does NOT address]

**Success** ✅
- Working Hypothesis: *"We believe that [doing X] will result in [outcome Y] for [user segment Z]"*
- Primary metric: [metric + threshold]
- Guardrail metrics: [what must not regress]

**🔍 Evidence**

Supporting Data ✅
[Pull from the project, GOALS.md metrics, meeting notes or docs. If none available: [Data needed]]

Competitive Research *(encouraged)*
[To complete]

Opportunity Sizing *(encouraged)*
[MAU reach, ARR impact — if known]

**⚠️ Dependencies, Risk & Mitigations**

Risks ✅
| Risk | Probability | Severity | Mitigation |
|------|-------------|----------|------------|
| [risk] | [H/M/L] | [H/M/L] | [action] |

Dependencies ✅
| Dependency | Type | Status |
|------------|------|--------|
| [dependency] | [Stakeholder/Technical/Calendar] | [action] |

---

#### 2. Solution Space *(complete after kickoff)*

**📝 Solution Definition**

Proposed Solution ✅
[What is the scope? Key features? Complexity?]

Rejected Options *(encouraged)*
[To complete]

Scope ✅
- In scope: [list]
- Out of scope: [list]

Key Features ✅
- Table stakes: [must-have baseline]
- Differentiators: [what sets this apart]

Product Vision *(encouraged)*
[To complete]

**🎨 Design/Prototype**

Interactive Prototype ✅
[Prototype link — To complete]

Design Explorations ✅
[Figma links — To complete]

Feasibility Review ✅
[To complete — schedule early]

Future Considerations ✅
[What's deferred to a future cycle]

**🧪 Eval Strategy** *(for AI-powered features)*

AI Task Definition
- Primary tasks: [what the AI needs to do]
- Inputs available: [data/context the AI has access to]
- Constraints: [hard rules]

Evaluation Levels
- Level 1 (basic checks): [To complete]
- Level 2 (human/model eval): [To complete]
- Level 3 (A/B): [To complete]

**🗺 Delivery**

Milestones ✅
| Milestone | Description | Target date |
|-----------|-------------|-------------|
| [M1] | [description] | [date] |

Critical Decisions ✅
[To complete]

Experiment & Rollout Plan ✅
- Primary metric + threshold: [metric]
- Guardrail metrics: [what can't break]
- Ramp gates: [To complete]
- Fallback: [To complete]

**📖 Definitions**
| Term | Definition |
|------|------------|

**🔗 References**
- [links to vision, strategy, decision docs, designs]

---

### Step 4 — Save to Context/Document Hub/

Save the full draft as:
`Context/Document Hub/[Initiative Name] PRD.md`

---

### Step 5 — Show in chat for review

Display the full PRD in chat, then show a compact summary:

```
📄 **PRD saved — [Initiative Name]**
Saved to: Context/Document Hub/[filename]
Project: [name or TBD]

**Sections completed:** [list required sections with real content]
**Needs your input:** [list [To complete] or [Data needed] sections]
```

Do NOT publish to the docs tool unless the user explicitly asks.

---

## Edge cases

- **No project found**: draft from GOALS.md + request context only. Flag clearly: *"No project found — create one and link after review."*
- **Pre-kickoff only**: draft Opportunity Space only. Note: *"Solution Space left blank — complete after kickoff."*
- **AI feature**: always include Eval Strategy section, even partially filled.
- **Existing doc found**: note it at the top — *"Existing doc found at [path] — this draft builds on it."*
