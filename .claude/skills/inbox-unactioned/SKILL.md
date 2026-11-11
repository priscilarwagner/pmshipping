---
name: inbox-unactioned
description: "Fetch and triage unanswered email — classify as Tonight (time-sensitive) or Tomorrow (can wait). Use when asked to check unread messages, triage the inbox, or see what needs attention."
---

Fetch and triage unactioned email for the relevant time period. Uses Gmail (see `Context/agent-config.md` → Tool Stack). If a comms tool is added to the config later, scan it the same way.

Steps:
- **Get the current date** — run `date +%Y-%m-%d` to determine today's date. Do NOT rely on the currentDate context block
- Determine the time period at hand (e.g. today's date if run standalone, or the day being summarised if run as part of a daily summary)
- Fetch threads from the relevant time period:
  1. Call Gmail `search_threads` with `in:inbox after:[YYYY/MM/DD] -from:me -category:promotions -category:updates -category:social -category:forums` and `view: THREAD_VIEW_SUMMARY` — the summary view shows the latest message per thread, so a thread whose latest sender is the user is already actioned
  2. Skip newsletters, receipts, notifications and other automated mail (`no-reply`, `noreply`, `notifications@`, `CATEGORY_PROMOTIONS`/`CATEGORY_UPDATES` labels) unless they need a decision or carry a deadline
  3. Skip calendar notifications — subjects starting `Invitation:`, `Updated invitation`, `Accepted:`, `Declined:`, `Appointment cancelled`, booking confirmations. The calendar already holds them
  4. Check `Context/Memory/` for any stored priority senders — always include their threads
- For each thread:
  1. Read the full thread with `get_thread`
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

Print the result directly. Do not write it to any file. Offer to draft replies; create them as Gmail drafts only, never send.
