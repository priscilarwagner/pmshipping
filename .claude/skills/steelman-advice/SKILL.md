---
name: steelman-advice
description: Multi-perspective steelmanning of any document using dynamically selected sub-agents. Returns blind spots, frameworks, and concrete improvements.
---

# /steelman-advice — Dynamic Multi-Perspective Steelmanning

Invoke: `/steelman-advice [doc]`

## What This Does

Analyzes any document (PRD, strategy doc, initiative, goal) and dynamically selects 3-5 critique perspectives based on what the document covers and — more importantly — what it's missing. Each perspective runs as a parallel sub-agent that searches for real-world evidence, frameworks, and concrete improvements.

The user walks out with:
- Aspects they haven't considered
- Concrete improvements with rationale
- Tangible frameworks to apply
- Specific challenges to their thinking

---

## Step 1: Clarify Before You Steelman

**Do not skip this step.** Before any analysis, you need answers to these four questions to scope the session:

1. **What is the actual decision this document supports?** Not the topic — the decision you're trying to make or get approved.
2. **What's your current lean?** Where are you landing, and what's your confidence level?
3. **What kind of challenge are you looking for?**
   - Blind spot hunt — "What am I missing?"
   - Stress test — "Where will this break?"
   - Alternative framing — "Is there a better approach entirely?"
   - Exec readiness — "Will this survive scrutiny from leadership?"
4. **What have you already pressure-tested?** What feedback have you already received, so perspectives don't repeat ground you've covered?

**Before asking:** Check whether the user already answered any of these — either in the prompt they typed when invoking the skill, or inside the document itself (e.g., a "Decision" or "Recommendation" section answers #1; a stated position answers #2; prior reviewer comments answer #4). Only ask the questions that remain unanswered.

If all four are already covered, skip straight to Step 2. If none are, ask them all at once.

Use the answers (from any source) to:
- Weight perspective selection in Step 4 (e.g., "exec readiness" favours Narrative Clarity and Stakeholder Alignment)
- Focus sub-agent analysis on the areas the user hasn't already covered
- Calibrate discomfort level (if they want a stress test, go harder; if they want blind spots, go wider)

If the user says "just run it," proceed with defaults: blind spot hunt, no prior feedback assumed, full perspective selection logic.

---

## Step 2: Load the Document

If a file path is provided, read it directly.

If only a name is provided, search common locations:
1. `Tasks/` directory (if it exists)
2. `Context/Document Hub/` directory (if it exists)
3. Use Glob with the name pattern across the workspace

Once found, read the full document.

---

## Step 3: Load Context

**Optional** (read if they exist, skip silently if they don't):
- `GOALS.md` — user's current priorities and success criteria (helps weight perspectives)
- `Context/Memory/` — user preferences, company context, learnings
- Any reference material in `Context/` that relates to the document's topic

---

## Step 4: Analyze Document and Select Perspectives

### 4a: Extract Document Profile

Read the document and identify:
- **Document type**: PRD, strategy doc, initiative brief, goal, roadmap, other
- **Key themes**: What topics does it cover? (growth, retention, technical architecture, team design, metrics, competitive positioning, etc.)
- **Stated goals/metrics**: What success looks like according to the author
- **Gaps**: What's conspicuously absent? (No competitive analysis? No failure criteria? No user research? No growth model?)

### 4b: Select 3-5 Perspectives from the Catalog

Pick perspectives that are **most relevant to the document's gaps and themes**. Follow these rules:

1. **Match 1-2 perspectives to the document's core theme** — to go deeper where it matters
2. **Match 1-2 perspectives to the document's gaps** — to surface what's missing
3. **Always include at least one "uncomfortable" perspective** — the one the author likely didn't want to hear
4. **Never select more than 5 or fewer than 3**

### Perspective Catalog

#### Craft

| Perspective | Lens | Best for |
|---|---|---|
| **Founder Mentality** | Are you building like an owner? Is this 10x or incremental? Would you bet your own money? | Initiative briefs, new product proposals |
| **Community-Centric Thinking** | Are real user needs driving this? Is the community voice present? Who's being left out? | PRDs, feature specs, UX proposals |
| **Product Sense** | Does the product intuition hold up? Are the trade-offs right? What would a great PM push back on? | PRDs, roadmaps, feature prioritization |
| **Technical Feasibility** | Is the technical approach sound? What are the hidden integration costs and scaling risks? | PRDs, technical proposals, architecture docs |

#### Strategy

| Perspective | Lens | Best for |
|---|---|---|
| **Long-Term Thinking** | Where does this lead in 2-3 years? What optionality are you killing? What are you locking in? | Strategy docs, roadmaps, platform decisions |
| **Ownership of Outcomes** | Are success metrics owned and measurable? Who's accountable? What does failure look like? | Goals, OKRs, initiative briefs |
| **Systems Thinking** | What are the second-order effects? Feedback loops? Unintended consequences across the ecosystem? | Strategy docs, platform changes, org proposals |
| **Decision Quality** | Are the key decisions reversible? Are you deciding with enough data? What's the cost of being wrong? | Any doc with major bets or commitments |

#### Communication

| Perspective | Lens | Best for |
|---|---|---|
| **Narrative Clarity** | Is the story compelling? Would an exec approve this in 5 minutes? Does it pass the "so what?" test? | Exec-facing docs, strategy proposals |
| **Stakeholder Alignment** | Who needs to buy in? What objections will surface? Are cross-functional dependencies named? | PRDs, roadmaps, initiative briefs |

#### Leadership

| Perspective | Lens | Best for |
|---|---|---|
| **Clarity from Chaos** | Is this cutting through ambiguity or adding to it? Is the path forward obvious after reading? | Strategy docs, complex proposals |
| **Team Empowerment** | Does this create agency or dependency? Can the team run with this without the author? | Org proposals, process docs, team charters |

#### Additional Lenses

| Perspective | Lens | Best for |
|---|---|---|
| **Growth & Retention** | What do growth loops, activation, and retention patterns look like? What's the growth model? | Growth strategies, activation plans, retention docs |
| **Competitive Moat** | Does this build defensibility? Can a competitor replicate this in 6 months? | Strategy docs, competitive analysis, platform proposals |
| **Metrics & Measurement** | Are the KPIs the right ones? Goodhart's Law risks? Vanity metrics hiding? | Goals, OKRs, experiment designs, PRDs |

---

## Step 5: Run Perspectives in Parallel

For each selected perspective, launch a parallel sub-agent. Each sub-agent receives:

### Sub-Agent Prompt Template

```
You are a product advisor reviewing a document through the lens of **[PERSPECTIVE NAME]**.

## Your Lens
[PERSPECTIVE LENS DESCRIPTION]

## The Document
[FULL DOCUMENT CONTENT]

## Your Task

1. Read the document carefully through your assigned lens.
2. Search the workspace for relevant context — `Context/`, `Tasks/`, any reference material.
3. Identify the observed bias or problem — what the document gets wrong, overlooks, or over-indexes on.
4. Formulate a concrete reframe — the new perspective the author should adopt instead.

## Output Format (follow exactly)

**Perspective: [PERSPECTIVE NAME]**

**Observed Bias / Problem:** [What the document gets wrong, overlooks, or over-indexes on. Be specific — name the section or claim. This is the uncomfortable part.]

**Reframe:** [The new perspective the author should adopt instead. Not just "consider X" — frame it as a concrete shift: "Instead of [current approach], try [alternative approach] because [reason]."]

**What to Improve:**
1. [Specific, actionable improvement tied to the reframe]
2. [Specific, actionable improvement tied to the reframe]

**Discomfort Rating:** [Low / Medium / High — how much this challenges the author's current thinking]
```

**Important:** Launch all sub-agents in parallel (they are independent). Do not launch more than 4 at a time — if 5 perspectives are selected, launch 4 first, then the 5th.

---

## Step 6: Consolidate Results

Wait for all sub-agents to complete, then consolidate into the Steelman Report.

### Output Format

```
# Steelman: [Document Name]
_Reviewed: [date] | Perspectives: [list of selected perspectives]_
_Why these perspectives: [1 sentence on selection rationale]_

## The Single Biggest Blind Spot
[Cross-perspective synthesis — the one theme that multiple perspectives flagged.
If 2+ perspectives independently identified the same gap, call it out here.]

## Perspective Summaries

| Perspective | Observed Bias / Problem | Reframe | Discomfort |
|---|---|---|---|
| [Name] | [1-line bias/problem] | [1-line reframe] | [Low/Med/High] |
| ... | ... | ... | ... |

## Perspectives & Reframes

### [Perspective Name]

**Observed bias / problem:** [What the document gets wrong or over-indexes on — be specific, name the section or claim]

**Reframe:** [The concrete shift in perspective. Not "consider X" but "Instead of [current approach], try [alternative] because [reason]."]

**What to improve:**
1. [Actionable improvement tied to this reframe]
2. [Actionable improvement tied to this reframe]

---

[Repeat for each perspective, separated by horizontal rules]

## What to Improve (Consolidated)
_Ordered by impact, deduplicated across perspectives._

1. [ ] [Improvement] — flagged by [perspective(s)]
2. [ ] [Improvement] — flagged by [perspective(s)]
3. [ ] [Improvement] — flagged by [perspective(s)]
```

---

## Step 7: Offer Follow-Up Actions

After presenting the consolidated output, offer:

1. **Go deeper on a perspective:** "Want me to expand on [perspective]'s findings?"
2. **Draft revised sections:** "Want me to draft the revised [section name] incorporating the feedback?"
3. **Run with different perspectives:** "Want me to re-run with [other perspective] instead of [current one]?"
