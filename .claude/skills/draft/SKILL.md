---
name: draft
description: "Universal drafting router — detects what type of draft is needed, uses the correct template, and shows output in chat for review. Handles goal updates, PRDs, meeting minutes, emails, short messages, and strategy docs."
---

# Draft

Detect the type of content needed, apply the right template, draft it in the user's voice, and show the result in chat. Nothing is sent or published unless the user explicitly asks.

---

## Step 1 — Detect draft type

Read the request and classify into one of these types:

| Type | Signals |
|------|---------|
| `goal_update` | A Linear project or goal mentioned; "goal", "progress update", "status update", "project update", "monthly update" |
| `prd` | "PRD", "product requirements", "initiative brief", "kickoff doc", "feature spec", "product brief" |
| `meeting_minutes` | "meeting notes", "minutes", "recap", "notes from [meeting]", "what was discussed", "write up the meeting" |
| `email` | "email", "reply to [person]", "write to [person]", "message to [email]" |
| `short_message` | "message", "DM", "text", "post in", "LinkedIn message", "send to [person]" (not email) |
| `strategy_doc` | "strategy", "memo", "write up", "one-pager", "narrative", "doc about" |

If the type is ambiguous, ask one question: *"What type of content is this — goal update, PRD, meeting notes, email, short message, or something else?"*

---

## Step 2 — Route to the correct sub-skill

Read the corresponding skill file and execute it fully:

| Type | Skill file |
|------|-----------|
| `goal_update` | `.claude/skills/draft-goal-update/SKILL.md` |
| `prd` | `.claude/skills/draft-prd/SKILL.md` |
| `meeting_minutes` | `.claude/skills/draft-meeting-minutes/SKILL.md` |
| `email` | Execute inline — see Email instructions below |
| `short_message` | Execute inline — see Short message instructions below |
| `strategy_doc` | Execute inline — see Strategy Doc instructions below |

Pass the full original request as context when executing the sub-skill.

---

## Inline: Email

**Gather context:**
- Check `Context/Memory/Reference/writing-styles/` for voice guide
- Check `Context/121s/` if the recipient has a relationship doc
- Check `Context/Meeting Notes/` for relevant recent context

**Draft rules:**
- Lead with the point, not a greeting
- 2–3 sentences per paragraph max
- Specific and direct — no filler phrases ("I hope this finds you well", "just wanted to", "as per my last email")
- Close with a clear ask or next step
- Sign off: the short name set in `Context/agent-config.md` → Name (first name only, no formal sign-off block)

**Show in chat:**
```
✉️ **Email draft — to [recipient] re: [subject]**
Subject: [suggested subject]

[Full email text]
```
Say "create a draft" to save it as a Gmail draft, or "send it" to send.

---

## Inline: Short message

For chat messages, DMs, LinkedIn messages or comments — anything short that isn't email.

**Gather context:**
- If replying in a known thread, ask the user to paste it for tone and context
- Check `Context/121s/` if directed at a specific person

**Draft rules:**
- No subject line — lead straight into the message
- Conversational but not sloppy
- Use bullet points only if listing 3+ distinct items
- Avoid excessive emoji — use 1 if it fits, not as decoration
- Keep it to what fits on a phone screen

**Show in chat:**
```
💬 **Message draft — [where / to whom]**

[Full message text]
```
Copy and paste it where it needs to go.

---

## Inline: Strategy doc / one-pager

**Gather context:**
- Read `GOALS.md` for strategic context
- Check `Context/Document Hub/` for related docs
- Check `Context/Memory/Reference/frameworks/` for relevant frameworks (JTBD, 7 Powers, etc.)

**Structure:**
```markdown
# [Title]
*[Date] | [Author: name from agent-config]*

## TL;DR
[2–3 sentences. What is this, why does it matter, what does it recommend.]

## Background
[Context — what's the situation, what led to this?]

## The Opportunity / Problem
[What we're solving and why now.]

## Proposed Approach
[What we're recommending. Be specific.]

## Why This, Not That
[Key tradeoffs or alternatives considered and rejected.]

## Success Looks Like
[Concrete outcomes — metric + timeframe where possible.]

## Open Questions
[What still needs to be resolved before moving forward.]

## Next Steps
| Action | Owner | By when |
|--------|-------|---------|
```

**Save to:** `Context/Document Hub/[title].md`

**Show in chat:**
```
📄 **Strategy doc saved — [title]**
Saved to: Context/Document Hub/[filename]

[TL;DR text]
```
Say "move to Notion" to publish it there.

---

## Voice rules (apply to all types)

Read `Context/Memory/brain.md` before drafting. Apply these always:
- Lead with outcome, not effort or process
- Specific over vague — name the thing
- Short sentences — no padding
- Confident, not hedging
- No generic AI phrasing ("it's worth noting", "in conclusion", "this is important because")
- Internal docs: casual but precise. External: professional but not stiff.

---

## Output rule

All draft types show output in chat. Never send, post or publish automatically — only act if the user explicitly asks (e.g. "send it", "create a Gmail draft", "move to Notion").
