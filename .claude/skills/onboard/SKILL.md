---
name: onboard
description: "Run a structured onboarding interview to populate GOALS.md with goals and profile context"
---

# Onboard

Guide the user through a structured interview to populate `GOALS.md` with goals, priorities, and profile context.

## Instructions

When the user invokes `/onboard`, run through all steps in sequence.

### Step 0: Tool Stack & MCP Connection Check

Before anything else, ask the user which tools they use. Ask all in a single message:

> **Quick setup — which tools do you use?**
>
> 1. **Project management** — where do you track tasks and issues? *(e.g. Linear, Jira, Asana, GitHub Issues)*
> 2. **Email & calendar** — *(e.g. Gmail + Google Calendar, Outlook)*
> 3. **Docs** — where do your long-lived docs live? *(e.g. Notion, Confluence, Google Docs)*
> 4. **Comms** — team messaging tool, if any? *(e.g. Slack, Teams — or none)*
> 5. **Meeting notes** — where do meeting notes or transcripts come from, if anywhere? *(e.g. Notion meeting notes, a recorder, or none — paste manually)*

Wait for their response. These go into the **Tool Stack** table in `Context/agent-config.md` in Step 6. All skills read that table.

The skills in this repo are written for **Linear, Gmail, Google Calendar and Notion**, with no comms tool. If the user names a different tool, note that the matching skill steps need adapting or will run on pasted context.

Then check which MCPs are connected. Probe only the ones relevant to their stated tools:

| Role | MCP | Probe call | What it unlocks |
|---|---|---|---|
| **Project management** | Linear | `list_issues` with `assignee: "me"`, limit 1 | achievements, today, plan-week, morning-briefing, draft-prd, draft-goal-update, draft-idea |
| **Email** | Gmail | `search_threads` with `in:inbox`, limit 1 | inbox-unactioned, morning-briefing, today, 121, achievements |
| **Calendar** | Google Calendar | `list_calendars` | morning-briefing, today, meeting-digest, plan-week |
| **Docs** | Notion | `notion-search` with a simple query | draft-prd, draft-experiment-doc, draft-reqs, meeting-digest, update-graph |
| **Files** | Google Drive | `list_recent_files`, limit 1 | meeting-digest, draft-meeting-minutes, today (Gemini meeting notes) |

For each probe: if the call succeeds or returns results → ✅ connected. If it returns an auth error, tool-not-found, or times out → ❌ not connected.

Present the results as a status table, naming the tool they mentioned:

```
MCP connection check:

✅ Linear (project management)
✅ Gmail (email)
✅ Google Calendar
❌ Notion (docs)
— Comms: none
```

**If all relevant MCPs are connected:** confirm and continue to Step 1.

**If any are missing:** show the setup note below, then ask:

> "Connect any missing MCPs and come back, or continue now and those skills will be limited."

Wait for their choice — if they say continue, proceed to Step 1 and note which skills will be limited at the end (Step 7).

---

#### How to connect MCPs

In the Claude desktop app or claude.ai, open **Settings → Connectors** and add Linear, Gmail, Google Calendar and Notion. Each signs in through the browser. In the Claude Code CLI, run `/mcp` to check what's connected.

After connecting, restart the session and re-run `/onboard` to re-check.

---

### Step 1: Check Current State

1. Find the goals file: if `Context/agent-config.md` exists, use its **Goals File** path; otherwise use `GOALS.md` at the root. Every mention of `GOALS.md` below means that path. Read it (if it doesn't exist, treat it as blank)
2. Determine whether it's still a blank template:
   - **Blank/template**: contains placeholder text — strings like `[Your role`, `[Date]`, `[Priority 1]`, `[placeholder]`, etc.
   - **Already filled**: real content has replaced the placeholders
3. If already filled:
   - Tell the user: "Looks like you're already set up. Want to review and refresh your goals instead of starting over?"
   - If yes: proceed with the interview but frame it as a refresh, pre-populate answers from current values, and confirm changes before writing
   - If no: stop
4. If blank or template: proceed to Step 2

### Step 2: Choose a path

Ask the user which approach they'd like in a single message:

> **Two ways to get set up — pick whichever feels easier:**
>
> **Option A — Quick bootstrap via another AI assistant:**
> Paste the following prompt into ChatGPT or Claude (with memory turned on for best results). Copy the output and paste it back here — I'll take it from there.
>
> ---
> *I'd like you to bootstrap the following personal data about me by looking at my biggest documents, meetings and conversations over the past 3 months. Follow this exact template and answer the questions:*
> - *What's your current role, and what company/team are you on?*
> - *What's your primary professional vision? What are you building toward?*
> - *In 12 months, what would make you think 'this was a successful year'?*
> - *What's your 5-year north star? Where do you want to be?*
> - *What are you actively working on right now?*
> - *What are your objectives for THIS QUARTER (next 90 days)?*
> - *What skills do you need to develop to achieve your vision?*
> - *What key relationships or network do you need to build?*
> - *What's currently blocking you or slowing you down?*
> - *What opportunities are you exploring or considering?*
> - *Any working style preferences or constraints worth noting?*
> ---
>
> **Option B — Answer my questions directly** (2 quick rounds, takes about 5 minutes)

Wait for the user's choice before continuing.

- If they choose **Option A** (or paste output): go to Step 3A
- If they choose **Option B**: go to Step 3B

### Step 3A: Bootstrap from AI output

1. Accept the pasted output from ChatGPT or Claude
2. If any connected tools are available (Notion, Linear, Gmail), search them for context that enriches the answers — e.g. recent project docs, OKRs, or threads that add depth to what was pasted
3. Synthesize everything into a `GOALS.md` draft, then go to Step 4

### Step 3B: Interview — Round 1 (Who you are & where you're going)

Ask all of the following in a single message:

> **A few questions to get you set up** — answer as much or as little as you like:
>
> 1. What's your current role, and what are you primarily building or working on?
> 2. What company or team are you on? Who do you report to? *(Optional)*
> 3. What's your professional north star — where are you trying to get in the next year?
> 4. What would make this year feel like a genuine success? Be specific if you can.
> 5. *(Optional)* Where do you want to be in 5 years?

Wait for the user's response, then ask Round 2:

> **One more round — your current priorities:**
>
> 1. What active projects or initiatives are you working on right now?
> 2. What are your objectives for the next 90 days? (3–5 concrete, measurable outcomes)
> 3. What are your top 3 priorities right now — the things that matter most this week and month?
> 4. What's currently blocking you or slowing you down?
> 5. *(Optional)* Any skills you want to build, key relationships to develop, or working style preferences worth noting?

Wait for the user's response, then go to Step 4.

### Step 4: Draft & Confirm

1. Synthesize all answers into a fully populated `GOALS.md` draft
2. Use the existing `GOALS.md` structure — replace every placeholder with the user's actual answers
3. Add a **Profile** section at the top (after "Last updated") with any facts that don't fit elsewhere:
   - Company / Team (if provided)
   - Reporting structure (if provided)
   - Working style notes or preferences (if mentioned)
   - Omit fields the user didn't provide
4. Set "Last updated" to today's date
5. **If refreshing** (GOALS.md was already filled): merge new answers into the existing content — preserve anything not covered in this session, update fields that were answered, add new fields where missing. Do not erase existing content the user did not revisit.
6. Present the full draft to the user as a readable preview
7. Ask: *"Does this capture it accurately? Any changes before I save?"*
8. Wait for confirmation. Apply any corrections before writing.

### Step 5: Write GOALS.md

On confirmation, write the final draft to the goals file path from Step 1.

### Step 6: Set up agent-config

After writing GOALS.md, check whether `Context/agent-config.md` already exists.

- **If it exists:** read it, show the current values, and ask: *"Your agent-config looks like this — anything to update?"* Apply any changes they mention.
- **If it doesn't exist:** run the setup interview below, then write the file, starting from `Context/agent-config.example.md`.

**Agent-config setup interview** — ask all questions in a single message:

> **One more thing — a few settings so all skills work out of the box:**
>
> 1. What's your timezone? *(e.g. Australia/Sydney, Europe/London, America/New_York)*
> 2. What's the full path to this workspace on your machine? *(e.g. /Users/yourname/Agents/pmshipping)*
> 3. Which Linear workspace (and default team) should I use?
> 4. For PRDs and experiment docs — what name should appear as the default driver?
> 5. Who is your default approver? *(Optional — skip if not applicable)*

Wait for their response, then write `Context/agent-config.md` using the structure of `Context/agent-config.example.md`. Fill the **Tool Stack** table from Step 0. Populate only the fields they provided — leave others with a `[To fill in]` placeholder.

### Step 7: Close

Tell the user what was written and suggest next steps — keep it brief:

> "Done. `GOALS.md` and `Context/agent-config.md` are set up.
>
> Next steps:
> - `/today` — build your first daily plan
> - Drop items into `BACKLOG.md` and run `/backlog` to turn them into goal-aligned tasks
> - Optional: create a review-cycle goals file in `Context/` (e.g. `Context/Goals H1 2027.md`) and add its path to agent-config — `/achievements` and `/draft-goal-update` read from it"

If any MCPs were skipped in Step 0, append:

> "⚠️ Some skills are limited until you connect the missing MCPs:
> - **No Linear** → achievements, today, plan-week, morning-briefing and the draft skills won't query your issues and projects
> - **No Gmail** → inbox-unactioned, morning-briefing and 121 won't pull email context
> - **No Google Calendar** → morning-briefing, today, meeting-digest and plan-week won't include your schedule
> - **No Notion** → doc skills won't read existing docs or log decisions to Notion
>
> Re-run `/onboard` after connecting them — it'll skip the parts you've already done."

## Output Format

- Batch questions into exactly 2 rounds — never ask one question at a time
- Show the full `GOALS.md` draft as a readable preview before writing anything
- Never write files without explicit user confirmation
- Keep the closing message brief — no walls of text
