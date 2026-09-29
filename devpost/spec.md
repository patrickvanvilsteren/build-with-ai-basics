---
doc: spec
status: approved
---

# Wild Wolves Agenda Bridge — Technical Spec

## How This Works, In Plain Language
The whole app is **one Python program that you start by hand** in a terminal. When you run it, it does three things in a row:

1. It asks **Discord** "which scheduled events does the Wild Wolves server have in the next 7 days?" Discord only answers programs it knows, so the program identifies itself as a **bot** (a small automated account added to the server, with almost no rights, only able to look).
2. It asks **Outlook** "what is already on my calendar in that same 7 days?" and throws away every Discord event that is already there (same name **and** same date/time).
3. For each event left over, it tells Outlook "add this as a **tentative** item" with the title, time, description and location, and prints the list on screen.

Then you open Outlook and decide what to keep.

Talking to Discord and Outlook is done by sending **web requests**: like submitting a form to someone else's building and reading their reply. Discord's reply and Outlook's reply are both structured data (JSON) that Python can read like a list of labelled boxes.

Nothing is saved between runs by the program itself. Outlook *is* the memory: because it checks Outlook every time, running twice never creates duplicates.

Why this shape: a single script with three small helper files is the smallest thing that proves "a Discord event turns into a tentative Outlook item". No website, database, login screen, or scheduler is needed for that.

## The Core Journey Through the System
PRD ref: `prd.md > The Core Journey`.

1. **You run** `python main.py` in a terminal. → `prd.md > Run on demand`. The program starts, does one check, and ends.
2. **Settings load** from your private `.env` file (bot token, server ID, Microsoft app ID). Nothing is typed into chat or committed.
3. **Discord is asked** for the server's scheduled events (`discord_events.py`). The answer is filtered to events starting between *now* and *now + 7 days*. → `prd.md > Finding events (next 7 days)`.
4. **Outlook is asked** for your calendar items in that same window (`outlook_calendar.py`). The first time ever, a browser window opens so you sign in to Microsoft and give permission; later runs reuse a saved sign-in.
5. **Comparing** (`main.py`): an event is "already there" when an Outlook item has the same name and same start time. Those are dropped silently. → `prd.md > Skipping events already in Outlook`.
6. **Creating**: each remaining event is sent to Outlook as a tentative item. → `prd.md > Creating tentative Outlook items`.
7. **Printing**: the list of new events (name, date/time), or "no new events found", or a "not available" message. → `prd.md > Result list`, `prd.md > States and Boundaries`.
8. **You open Outlook**, see the tentative items, and keep or remove them.

```
python main.py
   │
   ├──> Discord  (bot token) ──> events in next 7 days
   ├──> Outlook  (your login) ──> items in next 7 days
   ├──  compare: same name + same start time? → skip
   └──> Outlook  create tentative item  ──> print list
```

## Stack
- **Python 3.11 or newer** — chosen by you (matches your goal of learning basic Python). Docs: https://docs.python.org/3/
- **`requests`** — plain web requests to Discord and Microsoft Graph; recommended and accepted because every step stays visible (tradeoff: slightly more hand-written code than a Discord library). Docs: https://requests.readthedocs.io/
- **`msal`** — Microsoft's small library for the sign-in step; hand-writing the login is error-prone and not educational. Docs: https://learn.microsoft.com/entra/msal/python/
- **`python-dotenv`** — reads the private `.env` settings file. Docs: https://pypi.org/project/python-dotenv/
- **`pytest`** (optional, small) — one or two tests for the "is this a duplicate?" rule. Docs: https://docs.pytest.org/

Not verified yet (no lookup done while planning; check at the start of the build): current exact versions of `msal` and `requests`, and that Python 3.11+ is what is installed on your laptop.

## Where It Runs and How Someone Tries It
- **Runs locally** on your Windows laptop, in a terminal. Chosen by you; no deployment for the PoC.
- **Requirements:** Python 3.11+, `pip install -r requirements.txt`, a filled-in `.env`, and a one-time Microsoft sign-in in the browser.
- **Start command:** `python main.py` (from the project folder).
- **Demo recording:** show (1) the Discord server's scheduled events, (2) the terminal run listing the new events, (3) Outlook with the new tentative items, (4) a second run printing "no new events", and (5) you keeping one item and removing another.
- **Submission requires** a short demo video **and** a public GitHub repository. The repo contains `.env.example` (setting names only), never the real `.env`, the Discord token, or the Microsoft token cache. Deployment is not planned; it can be reconsidered in `6-ship`.

## Look and Feel
Non-visual tool (`prd.md > Look and Feel`). Plain, short terminal output: one line per new event, e.g. `Sat 04 Oct 20:00  Game Night`, plus clear one-line messages such as `No new events found.` and `Discord is not available. Nothing was created.` No colors or decoration required.

## Components

### Entry point and comparison — `main.py`
Runs the steps in order: load settings, get Discord events, get Outlook items, drop duplicates, create tentative items, print results. Contains the duplicate rule: same name (ignoring capital letters and extra spaces) **and** same start time (compared in UTC, to the minute). Catches "not available" problems and prints the plain message.
PRD ref: `prd.md > Run on demand`, `prd.md > Skipping events already in Outlook`, `prd.md > Result list`, `prd.md > States and Boundaries`.

### Discord reader — `discord_events.py`
Fetches the server's scheduled events with the bot token and returns a simple list of events (name, start, end, description, location) that start within the next 7 days. Picks the location from the event's location text if it is an external event, or the voice/stage channel name otherwise.
PRD ref: `prd.md > Finding events (next 7 days)`.

### Outlook calendar — `outlook_calendar.py`
Handles Microsoft sign-in (first run opens a browser; later runs use the saved token file), lists calendar items in the 7-day window, and creates a tentative item.
PRD ref: `prd.md > Skipping events already in Outlook`, `prd.md > Creating tentative Outlook items`.

### Settings — `config.py`
Reads `.env` and stops with a clear message if something is missing.
PRD ref: `prd.md > States and Boundaries` (secrets never in chat or repo).

### Duplicate-rule tests — `tests/test_dedupe.py`
Tiny checks that same name + same time is skipped, and same name + different time is kept.
PRD ref: `prd.md > Skipping events already in Outlook`.

## Data Model
No database. Data moves through the program in memory during one run:

- **Discord event** (from Discord, lives only during the run): `name`, `start` (UTC), `end` (UTC or none), `description`, `location`.
- **Outlook item** (from Outlook, lives only during the run): `subject`, `start` (UTC).
- **Saved on disk:** only (a) `.env` (your settings) and (b) `.token_cache.json` (Microsoft sign-in token). Both are private and git-ignored. If they are missing, the program asks again or explains what to fill in.

Where data lives after a run and when you return: Outlook holds the created items; that is the "memory" that prevents duplicates.

## File Structure
```
Build with AI Basics/
├── main.py                 # start here: runs the whole check
├── discord_events.py       # asks Discord for scheduled events
├── outlook_calendar.py     # Microsoft sign-in, read items, create tentative item
├── config.py               # loads settings from .env
├── tests/
│   └── test_dedupe.py      # checks the duplicate rule
├── requirements.txt        # requests, msal, python-dotenv, pytest
├── .env.example            # setting names only (safe to publish)
├── .env                    # your real settings (git-ignored, never committed)
├── .token_cache.json       # Microsoft sign-in token (git-ignored)
├── .gitignore              # keeps the two private files out of GitHub
├── README.md               # setup + how to run + demo steps
└── devpost/                # Devpost learning workspace
```

## External Services and Dependencies

### Discord bot and API
- **What:** read scheduled events. Docs: https://discord.com/developers/docs/resources/guild-scheduled-event
- **Call:** `GET https://discord.com/api/v10/guilds/{guild_id}/scheduled-events` with header `Authorization: Bot <token>`.
- **Answer (list):** each event has `name`, `description`, `scheduled_start_time`, `scheduled_end_time`, `entity_type`, `entity_metadata.location` (external events), `channel_id` (voice/stage events), `status`.
- **Setup:** create an application + bot at https://discord.com/developers/applications (free), copy the token into `.env`, invite the bot to your **test server** yourself, and ask a Wild Wolves admin to invite it there (invite link only; no special powers). Server ID comes from Discord's Developer Mode → right-click server → Copy ID.
- **Cost / limits:** free; a run-by-hand script is far below rate limits.
- **To verify early:** exactly which bot permission is needed to see scheduled events (expected: just "View Channels"/basic access), and how recurring events appear in the list.

### Microsoft Graph (Outlook personal calendar)
- **What:** read and create calendar items. Docs: https://learn.microsoft.com/graph/api/resources/calendar
- **Sign-in:** register an app once at https://entra.microsoft.com (free; choose "personal Microsoft accounts" as the supported account type; allow public client flows). Use `msal` with authority `https://login.microsoftonline.com/consumers`, scopes `Calendars.ReadWrite` (offline access is handled by msal). Docs: https://learn.microsoft.com/entra/identity-platform/quickstart-register-app
- **Read:** `GET https://graph.microsoft.com/v1.0/me/calendarView?startDateTime=...&endDateTime=...&$select=subject,start` with header `Prefer: outlook.timezone="UTC"`. Docs: https://learn.microsoft.com/graph/api/user-list-calendarview
- **Create:** `POST https://graph.microsoft.com/v1.0/me/events` with `subject`, `body` (text: description), `start`/`end` (`dateTime` + `timeZone: "UTC"`), `location.displayName`, and `showAs: "tentative"`. If Discord gives no end time, use start + 1 hour. Docs: https://learn.microsoft.com/graph/api/user-post-events
- **Cost / limits:** free for personal accounts; far below limits.

## Important Failure Modes
- **Discord not reachable, wrong token, or bot not in the server** → prints `Discord is not available. Nothing was created.`; no Outlook changes.
- **Outlook sign-in fails or Graph is unreachable** → prints `Outlook is not available. Nothing was created.`; no partial creation (Outlook is read before anything is created).
- **Nothing new** (empty week or all duplicates) → prints `No new events found.`
- **A Discord event has no description or location** → the item is still created with those fields left empty.

## What Was Simplified and Why
- **Run by hand** instead of a daily 20:00 schedule — a schedule can't be shown in a short demo; the fuller version needs a scheduler (Windows Task Scheduler or a hosted job).
- **Outlook as the only memory** instead of a database or saved list of seen events — same duplicate protection with no extra storage.
- **Local run + recording** instead of hosting — the required video and repo don't need a live URL.
- **Test server first** instead of waiting on a Wild Wolves admin — proves the loop with the same code; only the server ID changes.
- **Changed events** (same name, moved date) are treated as new — updating existing items needs extra matching logic (from the PRD's open question).

## Decisions and Open Issues
**Decisions (your choices):**
- Outlook target is your **personal Outlook account**.
- **No use of your personal Discord login** (self-bots break Discord's terms and risk a ban); use a **bot** instead, and do **both**: your own test server and a request to a Wild Wolves admin.
- **Run locally** and record the demo.
- **Python script with plain `requests`** for Discord and **Microsoft Graph with `msal`** for Outlook — recommended by me, accepted by you.

**Implementation details derived from these (not learner decisions):** file layout above, 7-day window in UTC, end time defaults to start + 1 hour, `.env` for secrets.

**Your uncertainty and what clarified it:** you did not know a bot needs to be added by someone with rights in the server. Clarified in conversation (bot vs. self-bot, test-server fallback). Evidence during the build: the bot successfully lists the test server's events, and the same code works against Wild Wolves once the admin adds it.

**Still open / to check early in the build:**
- Accept/decline: Outlook's Accept/Decline buttons belong to meeting invitations from other people. On an item you create yourself, "accept" = change it from tentative to busy; "decline" = delete it. Confirm this in the demo and adjust the wording if needed.
- Which exact bot permissions are needed and how recurring Discord events are returned (see above).
- Whether a Wild Wolves admin agrees to add the bot (the demo works without it).
- Changed events (same name, new date) count as new (`prd.md > Open Questions`).
