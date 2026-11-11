---
name: bias
description: "On-demand bias audit — check a decision, plan, or situation against your motivational profile blind spots (supports Marlee, DISC, CliftonStrengths, Enneagram, or any scored assessment)"
---

# Bias Check

Run a targeted bias audit against your motivational profile from any assessment tool. Works standalone (full audit of current context) or with a specific topic.

## Instructions

When the user invokes `/bias [optional: topic or decision]`, run through all steps.

### Step 0: Check Profile Exists

Read `Context/Memory/bias.md`. If the file is missing or empty, run the **Setup Flow** below instead of the audit.

#### Setup Flow

Tell the user:

> **No motivational profile found.** I need your dimension scores to run bias checks. These come from any motivational or personality assessment tool — the most commonly used one with this system is [Marlee](https://getmarlee.com), but DISC, CliftonStrengths, Enneagram, or any scored framework works.
>
> **If you use Marlee:**
> 1. Go to [getmarlee.com](https://getmarlee.com) and sign in
> 2. Navigate to **People → Me** → click **All Motivations** to see the full bubble grid
> 3. Make sure all dimension categories are selected — click each filter pill or select-all
> 4. Screenshot the complete grid (you may need 2) and paste here
>
> **If you use another tool:**
> Export or screenshot your dimension scores and paste them here — include the dimension name and score/level for each.
>
> I'll extract your scores and set up your profile automatically.
>
> **No assessment?** Say "draft from my brain" and I'll build Sections B and C from the patterns already named in `Context/Memory/brain.md` (e.g. the breadth trap), marked as self-reported until you add scores.

When the user provides their results:

1. Extract every dimension name and score (accepting any format — screenshot, text export, copy-paste)
2. Create `Context/Memory/bias.md` using the **Profile Template** below
3. Derive communication rules (Section B) from the high/low patterns
4. Derive bias patterns (Section C) from the dimensional gaps
5. Confirm the profile is saved, then run the bias audit if the user provided a topic

#### Profile Template

Use the template structure — three sections:
- **Section A**: All dimensions sorted by score (highest first), grouped into tiers
- **Section B**: Communication rules derived from extreme scores (90+ high, sub-60 low)
- **Section C**: Bias patterns derived from high-low gaps that create blind spots

For deriving biases, look for these patterns:
- Very high score (90+) on one dimension with a low complementary dimension → blind spot
- Two high scores that reinforce each other → amplified bias (e.g., high Initiation + high Present = rushing to act)
- Low scores (sub-60) that create gaps in thinking or decision-making

### Step 1: Load Profile

Read `Context/Memory/bias.md` — load all bias patterns from Section C.

### Step 2: Load Context

If a topic was provided with the command (e.g., `/bias should I start a new research track`):
- Treat that as the subject of the audit

If no topic was provided:
- Read `Tasks/` for active and started tasks
- Read `GOALS.md` for current priorities
- Read recent meeting notes or daily notes
- Use the combined picture as the subject

### Step 3: Audit for Active Patterns

For each bias pattern in Section C, check whether it is present in the subject:

| Bias | Active? | Evidence |
|------|---------|---------|
| [pattern name] | Y/N | [specific signal if Y] |

Only show rows where Active = Y, unless no biases are detected (then say so clearly).

### Step 4: Recommend a Countermove

For each active bias, provide one concrete countermove — a specific next action that directly addresses the pattern. No generic advice.

Examples:
- **Breadth trap** → "Pick one track and define what 'deep enough' looks like before expanding."
- **Rushing to act** → "Spend 20 minutes mapping the second-order effects before deciding."
- **Initiation without assertion** → "Name who owns the outcome and what 'done' looks like by when."

### Step 5: Summary

One line: how many patterns are active, which is highest priority to address.

## Output Format

Lead with the table (active biases only). Follow with countermoves. Close with the one-line summary. Keep it under one screen.
