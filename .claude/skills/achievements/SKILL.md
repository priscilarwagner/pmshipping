---
name: achievements
description: "Weekly achievements log for performance review prep. Pulls official goal progress from Linear (projects, milestones and issues assigned to the user), surfaces extra work from meeting notes, email and docs, maps everything to the user's goals, flags growth behaviours, and appends a dated entry to a running achievements log. Use whenever someone says: /achievements, 'log my achievements', 'what did I deliver this week', 'update my review log', 'capture my wins', 'log my extra work', 'track what I shipped', 'review prep', or anything about recording weekly progress for review season."
---

# /achievements — Weekly Achievements Log

Build a weekly achievements entry that maps the user's work to their goals, flags growth behaviours, captures extra work beyond their official remit, and appends everything to a running log. The goal: self-review becomes a copy-paste exercise, not a memory test.

Run this standalone any day — Friday afternoon is ideal, but it works mid-week too.

---

## Configuration

At the start of the skill, read:
- `Context/agent-config.md` — for the goals file path
- The file referenced under **Goals File** in agent-config — for the user's current goals, goal tags, milestone hierarchy, growth behaviours, and routing rules

All goal descriptions, tags, milestone tables, and routing rules come from those files. Do not hardcode them here.

If there is no review-cycle file, use `GOALS.md`. If it has no goal tags, derive one kebab-case tag per quarterly objective (e.g. `#interview-ready`, `#portfolio`) and use its **Growth Goals** headings as the growth behaviour flags. Say which tags you derived so the user can keep or rename them.

---

## Step 1: Pull Official Goal Progress from Linear

Focus on what *moved* this week — status changes, comments, decisions — not just what exists.

**Issues assigned to the user, updated this week:**
`list_issues` with `assignee: "me"`, `updatedAt: -P7D`, ordered by `updatedAt`.

**Projects the user leads, updated this week:**
`list_projects` with `member: "me"` (or lead), then `get_status_updates` for any updated in the last 7 days.

**Milestones on those projects:**
`list_milestones` — note any that changed status this week.

For each result, extract: identifier, title, current status, project, and milestone. Use the project to determine which goal the work sits under — refer to the routing rules in the goals file. An issue moving to Done = achievement. An issue moving to In Progress = momentum.

---

## Step 2: Pull Extra Work Signals

Look for substantive work in the last 7 days that isn't tracked in Linear:
- **Meeting notes** — `Context/Meeting Notes/` files from the last 7 days: decisions the user drove, initiatives kicked off
- **Email** — Gmail `search_threads` with `from:me newer_than:7d`: decisions, coordination, things said no to
- **Docs** — Notion `notion-search` for pages the user created or edited this week, and `Context/Document Hub/` files modified this week

Look for:
- Decisions made or driven
- New initiatives being kicked off or scoped
- Capability building or tool adoption driven by the user
- Cross-functional coordination that isn't tracked in an issue
- Things the user said no to (deprioritisation decisions)

---

## Step 3: Synthesise Achievements

Write 4–8 entries. Quality over volume — a week with 5 sharp entries is better than 10 vague ones.

Use the goal tags and growth behaviour flags defined in the goals file.

**Entry format:**
```
**[Issue or project title] ([KEY-123](link))** `#[goal-tag]`

[Action taken — one line.]
[What was decided or delivered — one line.]
[Who was involved and what it unblocks — one line.]

[Optional: growth behaviour flag — one phrase, no explanation needed]
[Optional: → future goal candidate: [name]]
```

**Formatting rules:**
- Always name the issue or project title, with the key as a link, so it's clear what this maps to.
- Use line breaks between each fact — not full paragraphs, not a wall of prose.
- Lead with the action (what was done), then the outcome, then the names involved.
- Keep each line to one idea. If you need more than 4 lines, the entry is too long.
- Growth behaviour flag goes on its own line at the end, no explanation.
- Only flag a growth behaviour if the entry clearly shows it. Skip if it's a stretch.
- Only add `→ future goal candidate` if this topic has appeared in 2+ entries or has explicit stakeholder endorsement.
- Never invent metrics. If a number isn't in the sources, write `[metric?]`.

---

## Step 4: Ask for Additions

After presenting the synthesised entries, ask:

> "Anything I missed? Think about: decisions made that aren't in Linear, capability work done for the team, things explicitly said no to (visible prioritisation), extra coordination that isn't tracked anywhere, or new initiatives being seeded."

Wait for the user's response. Add anything they name with `#extra` and the appropriate goal tag(s). If they describe something that will likely become a formal goal, note it with a `→ future goal candidate` flag.

---

## Step 5: Append to the Achievements Log

Append to `Context/Progress Updates/achievements-log.md`.

If the file doesn't exist yet, create it with this header first:

```markdown
# Achievements Log

Running weekly record of review-relevant work. Built for self-review, goal tracking, and review season prep.

**Goal tags:** _(from current goals file)_

**Growth behaviour flags:** _(from current goals file)_

---
```

Then append the week's section (most recent at top):

```markdown
## CW[XX] — Week of [DD Mon YYYY]

[entries]

---
```

Confirm: *"Logged to `Context/Progress Updates/achievements-log.md`."*

---

## What Good Looks Like Over Time

After 8–10 weeks, the log should show:
- All goals represented most weeks — if one goes cold for 3+ weeks, flag it
- Growth behaviour flags appearing with increasing frequency
- `#extra` entries clustering around 1–2 repeating themes → these are next goal-setting proposals
- A trail of names, metrics, and decisions that can be cited verbatim in self-review

When `#extra` entries have appeared 3+ weeks in a row around the same theme, surface it: *"This pattern keeps showing up in #extra — worth considering as a formal goal next cycle."*
