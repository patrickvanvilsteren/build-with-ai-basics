---
doc: prd
status: approved
---

# Wild Wolves Agenda Bridge — Product Requirements

A small program you run by hand that finds new scheduled events in the Wild Wolves Discord server and adds them to your Outlook calendar as tentative items, so you can accept or decline them.
Source: `scope.md > The Core Loop`, `scope.md > The POC Boundary`.

## The Core Journey
1. You start the program yourself (there is no automatic schedule in the proof of concept). Source: `scope.md > The Core Loop`.
2. It looks at the scheduled events in the Wild Wolves Discord server that start within the **next 7 days from the moment you run it**.
3. For each event, it checks your Outlook calendar for an existing item with the **same name and the same date and time**. Matching events are skipped and not shown.
4. For each remaining (new) event, it creates a **tentative** Outlook calendar item using the event's title, date, time, description, and location.
5. It prints a list of the new events it found.
6. You open Outlook, see the tentative items, and accept or decline each one. Source: `scope.md > What "Working" Looks Like`.

Success: a Discord event you did not copy by hand shows up in Outlook as a tentative item with useful details.

## Screens and Layout
There is no separate app window or dashboard (`scope.md > Explicitly Cut`). Two surfaces:
- **The program's text output** where you start it and read the result list and messages.
- **Outlook**, where you review the tentative items and accept or decline them.

## Look and Feel
Non-visual tool. Output should be plain, short, and easy to read: one line or small block per event showing at least its name and date/time, and clear one-line messages for the empty and error cases.

## Features and Behavior

### Run on demand
- As the calendar owner, I want to start the check myself whenever I choose so that I stay in control.
  - [ ] Running the program once performs one full check and then ends.
  - [ ] No automatic or scheduled run exists.

### Finding events (next 7 days)
- As the calendar owner, I want only upcoming events in a short window so that I do not clutter my agenda with things far ahead.
  - [ ] Only events starting within the next 7 days from the time of the run are considered.
  - [ ] Events beyond 7 days are not listed or added.

### Skipping events already in Outlook
- As the calendar owner, I want events I already have to be ignored so that I only see new ones.
  - [ ] An event whose name **and** date/time match an existing Outlook item is not listed and not created again.
  - [ ] An event with the same name but a different date/time counts as new (for example next week's "Game Night").
  - [ ] Running the program twice in a row does not create duplicate items on the second run.

### Creating tentative Outlook items
- As the calendar owner, I want new events added as tentative items so that I can decide about each one.
  - [ ] Each new event appears in Outlook marked tentative.
  - [ ] The item carries the event's title, date, time, description, and location where Discord provides them.
  - [ ] I can accept or decline the item in Outlook and see it reflected in my agenda.

### Result list
- As the calendar owner, I want a list of what was found so that I can see what changed.
  - [ ] After a run with new events, the output lists each new event (name and date/time at minimum).
  - [ ] Events that were skipped as already existing are not shown.

## States and Boundaries
- **No new events** — the program prints a message that no new events were found. This covers both "nothing scheduled in the next 7 days" and "everything is already in Outlook".
- **Discord or Outlook not available** — the program prints a message that Discord / Outlook are not available, and creates nothing.
- **Scope of access** — only the Wild Wolves server is read; only your own Outlook calendar is written to. Logins and keys are never typed into chat or committed to the repository.

## Product Decisions
- Run by hand only for the proof of concept — a schedule cannot be shown in a short demo, and manual running proves the same idea. The daily 20:00 run moves to Later.
- Show a list of found events, and only new ones — the learner wants to see what changed, not repeats.
- A duplicate means same name **and** same date/time — so a weekly event with the same name still shows up each week.
- Look ahead 7 days from the moment of running — the learner may have other things planned beyond that.
- Plain messages for "no new events" and "Discord / Outlook not available".

## What We're Building
- A program started by hand.
- Reads scheduled events from the Wild Wolves Discord server, starting within the next 7 days.
- Skips events already in Outlook (same name and date/time).
- Creates tentative Outlook items with title, date, time, description, and location.
- Prints a list of new events, a "no new events" message, or a "not available" message.
- The learner accepts or declines the items in Outlook.

## Deferred From the POC
- **Daily automatic run at 20:00 (learner's time)** — needs a scheduler; can't be demonstrated in a one-minute video.
- **Looking further ahead than 7 days** — not needed to prove the loop.

## Possible Later Enhancements
- WhatsApp scanning and replying with the accept/decline decision.
- Email invitation handling.
- More Discord servers.
- Conflict handling with existing agenda items, reminders, and recurring events.

## Non-Goals
- Scanning ordinary Discord messages — scheduled events are a cleaner source.
- WhatsApp and email integrations — extra login and message complexity.
- Multiple servers — keeps the first experiment focused.
- A standalone interface or dashboard — Outlook is where you review and decide.

## Open Questions
- Whether a changed event (same name, moved date) should update the existing item — not covered; treated as a new event under the current rule. Can wait for the build.
