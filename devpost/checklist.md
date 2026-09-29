---
doc: checklist
status: draft
---

# Build Checklist

Build mode: not yet chosen

## Slices

- [ ] **1. The program lists your Discord server's upcoming events in the terminal**
  Becomes usable: Run `python main.py` and see the scheduled events of your Discord test server for the next 7 days, one line each (name and date/time).
  Why now: The riskiest unknown is the Discord bot (setup, permissions, how events and recurring events come back). Proving it first means later slices build on real data. Project scaffold, settings and dependencies live inside this slice.
  PRD ref: `prd.md > Run on demand`, `prd.md > Finding events (next 7 days)`, `prd.md > Result list`
  Spec ref: `spec.md > Components` (`main.py`, `discord_events.py`, `config.py`), `spec.md > File Structure`, `spec.md > External Services and Dependencies > Discord bot and API`
  Build: Create `requirements.txt`, `.env.example`, `config.py`, `discord_events.py` (GET scheduled events, keep those starting in the next 7 days, pick location from external location or channel name) and a `main.py` that prints one line per event. Check the installed Python version and current `requests` version. Confirm `.gitignore` covers `.env` and `.token_cache.json`.
  Verify (mechanical): Run `python main.py` against the test server with at least one scheduled event inside 7 days and one beyond; confirm only the in-window event is printed with the right name and time. Confirm a wrong token gives a clear message instead of a crash.
  Learner check: Create a scheduled event in your test server, run `python main.py`, and confirm the event shows up with the right name and time (and that an event 10 days out does not).
  Commit: `Read scheduled events from Discord`

- [ ] **2. The program signs in to your Outlook and reads your calendar for the next 7 days**
  Becomes usable: `python main.py` also prints how many items are on your personal Outlook calendar in the next 7 days, after a one-time browser sign-in that is then remembered.
  Why now: Microsoft sign-in (app registration, `msal`, token file) is the second big unknown. Proving read access before writing anything keeps your calendar safe and surfaces setup problems early.
  PRD ref: `prd.md > Skipping events already in Outlook`, `prd.md > States and Boundaries`
  Spec ref: `spec.md > Components` (`outlook_calendar.py`), `spec.md > External Services and Dependencies > Microsoft Graph`, `spec.md > Data Model` (`.token_cache.json`)
  Build: Add `outlook_calendar.py` with `msal` sign-in (consumers authority, `Calendars.ReadWrite`, token cache file) and a function that lists calendar items (`subject`, `start` in UTC) in the 7-day window. Print the item count from `main.py`. Add `msal` and `python-dotenv` versions to `requirements.txt`.
  Verify (mechanical): Run `python main.py`; confirm sign-in completes, the token file is created and git-ignored, and a second run signs in silently. Compare the printed items with a known item you put on your calendar.
  Learner check: Run it, sign in when the browser opens, and confirm the item count matches what you see in Outlook for the next 7 days.
  Commit: `Sign in to Outlook and read calendar items`

- [ ] **3. A Discord event shows up in Outlook as a tentative item (the kernel)**
  Becomes usable: Run the program and each Discord event of the next 7 days appears in your Outlook as a tentative item with title, date, time, description and location. (Running twice would still create duplicates until slice 4.)
  Why now: This is the unique kernel — an event you did not copy by hand becoming a tentative calendar item. It comes as soon as both sides are proven.
  PRD ref: `prd.md > Creating tentative Outlook items`, `prd.md > The Core Journey` (steps 4-6)
  Spec ref: `spec.md > Components` (`outlook_calendar.py`), `spec.md > External Services and Dependencies > Microsoft Graph` (Create)
  Build: Add a `create_tentative_event` function to `outlook_calendar.py` (POST `/me/events`, `showAs: "tentative"`, UTC start/end, end defaults to start + 1 hour, empty description/location allowed). Call it from `main.py` for each Discord event and print the list.
  Verify (mechanical): Run once with two test events (one with description and location, one without); confirm via a Graph read-back that both items exist with `showAs` tentative and the right times and fields. Delete the test items afterwards.
  Learner check: Run it and open Outlook. Find the tentative items, check title/time/details, then try to keep one (accept = mark as busy) and remove another (decline = delete), and say whether that works the way you pictured.
  Commit: `Create tentative Outlook items from Discord events`

- [ ] **4. Events already in Outlook are skipped, so running twice creates nothing new**
  Becomes usable: A second run prints "No new events found." and adds no duplicates; next week's same-named event still counts as new.
  Why now: Duplicates are only a problem once creating works, and the rule needs the read side from slice 2 and the create side from slice 3.
  PRD ref: `prd.md > Skipping events already in Outlook`, `prd.md > States and Boundaries` (no new events)
  Spec ref: `spec.md > Components` (`main.py` duplicate rule, `tests/test_dedupe.py`), `spec.md > Data Model`
  Build: Add the duplicate rule to `main.py` (same name ignoring case and extra spaces, and same start time to the minute in UTC), drop matches before creating, print `No new events found.` when nothing is left, and add `tests/test_dedupe.py` covering skip vs keep.
  Verify (mechanical): Run `pytest` and confirm the duplicate tests pass; run `python main.py` twice and confirm the second run creates nothing and prints the no-new-events message; confirm a same-name event at a different time is still created.
  Learner check: Run the program twice in a row and confirm Outlook has no duplicates, then add a same-named Discord event on another day and confirm only that one is added.
  Commit: `Skip events that already exist in Outlook`

- [ ] **5. Clear messages when Discord or Outlook is unavailable, plus setup README**
  Becomes usable: A broken token, missing setting or failed sign-in gives a plain one-line message and creates nothing; the README explains setup and how to try the app.
  Why now: Failure handling comes last because it only wraps behavior that already works; the README is needed for the public repo and demo.
  PRD ref: `prd.md > States and Boundaries`
  Spec ref: `spec.md > Important Failure Modes`, `spec.md > Where It Runs and How Someone Tries It`, `spec.md > Look and Feel`
  Build: In `main.py` catch Discord and Outlook failures and print `Discord is not available. Nothing was created.` / `Outlook is not available. Nothing was created.` (Outlook is read before anything is created); make `config.py` report missing settings clearly; write `README.md` (setup, `.env`, first sign-in, run, demo steps).
  Verify (mechanical): Run with a deliberately wrong Discord token and with the network-blocked Outlook token cache removed/invalid to confirm each message and that nothing was created; run `pytest` and a normal run to confirm nothing regressed.
  Learner check: Break the Discord token in `.env` on purpose, run the program, and confirm you get the plain "not available" message; restore it and confirm a normal run works.
  Commit: `Handle unavailable services and add README`

## Hands-on Checkpoints

- [ ] Early usable behavior explored — after slice 3 (first real Discord event turned into a tentative Outlook item)
- [ ] Final kick-the-tires exploration and feedback completed

## Final Review

- [ ] Final review complete — feedback resolved and learner confirms ready to ship

## Code Tour and App Map

- [ ] Learning activity complete — guided route, focused alternative, prior practice connected, or brief recap
- [ ] Optional edit and transfer reflection addressed — offered/declined/already covered/not applicable as appropriate
- [ ] `devpost/app-map.html` generated from finished code, checked, and shown, including a project-grounded practice to reuse

Activity and evidence: not started
Route and stops: not started
Edit outcome: not started
Reflection: not started
Activity mode: not started

## Revisions
