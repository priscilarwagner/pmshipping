---
name: inbox-unactioned
description: "Fetch and triage unanswered email — classify as Tonight (time-sensitive) or Tomorrow (can wait). Use when asked to check unread messages, triage the inbox, or see what needs attention."
---

> **Tools:** steps call role operations (`pm.*`, `email.*`, `calendar.*`, `docs.*`, `files.*`, `comms.*`). Resolve each through `Context/agent-config.md` → **Tool Stack** and the adapter it names (`.claude/adapters/README.md`). A role set to `none` → skip that step.

Fetch and triage unactioned messages for the relevant time period: email, plus `comms.mentions` if a comms tool is configured.

Steps:
- **Get the current date** — run `date +%Y-%m-%d` to determine today's date. Do NOT rely on the currentDate context block
- Determine the time period at hand (e.g. today's date if run standalone, or the day being summarised if run as part of a daily summary)
- Fetch threads from the relevant time period:
  1. `email.inbox(since)` — the adapter already strips newsletters, automated mail and calendar notifications. A thread whose latest message is from the user is already actioned
  1b. If a comms tool is configured: `comms.mentions(since)`
  2. Skip newsletters, receipts, notifications and other automated mail (`no-reply`, `noreply`, `notifications@`, `CATEGORY_PROMOTIONS`/`CATEGORY_UPDATES` labels) unless they need a decision or carry a deadline
  3. Skip calendar notifications — subjects starting `Invitation:`, `Updated invitation`, `Accepted:`, `Declined:`, `Appointment cancelled`, booking confirmations. The calendar already holds them
  4. Check `Context/Memory/` for any stored priority senders — always include their threads
- For each thread:
  1. Read the full thread (`email.thread` / `comms.thread`)
  2. Check whether the user has already replied — if the last message is from the user, it is actioned and can be skipped
  3. Refer to active tasks in `Tasks/` and goals in `GOALS.md` to understand context and priority
  4. Classify unactioned threads as **Tonight** (time-sensitive: someone is blocked, a decision is needed for something imminent, a deadline in the next 24 hours, or a direct ask from a key contact) or **Tomorrow** (everything else: FYIs, async updates, questions that can wait overnight)
  5. Capture a one-line summary and the sender

Format the output as two groups:

**Tonight**
- [thread summary] — [sender]

**Tomorrow**
- [thread summary] — [sender]

If there are no items in a group, omit that group entirely.

Print the result directly. Do not write it to any file. Offer to draft replies; create them with `email.draft` only — never send.
