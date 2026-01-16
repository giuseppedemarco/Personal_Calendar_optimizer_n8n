# Personal Calendar Optimizer (n8n + Notion + Ollama + Telegram)

An n8n workflow that reads tasks/events from a Notion Inbox, uses **Ollama (local LLM)** to extract structured fields, finds the best available time slot in your Notion Calendar, and sends confirmations via **Telegram (only)**.

## What this workflow does

### 1) Inbox → Calendar auto time-blocking (runs every minute)
- Watches a Notion **Inbox** database (polling every minute).
- Reads the page title (Name) and asks **Ollama** to extract:
  - `title` (string)
  - `duration` (minutes)
  - `priority` (1–10)
  - `type` (e.g., “Deep work”, “Meeting”, “Health”, “Personal”)
  - `flex` (“Flexible” / “Fixed”)
  - `preferredTime` (HH:MM if present)
  - `date` (YYYY-MM-DD, defaults to today)
  - `buffers` `{ before, after }` (minutes)
- Queries your Notion **Calendar** for events on the *target date only*.
- Generates 15-min slots inside predefined working windows:
  - Mon–Fri: 07:30–09:00, 12:30–14:00, 18:00–22:30
  - Sat–Sun: 10:00–13:00, 17:00–21:00
- Scores slots (priority-weighted + deep work morning bonus + health evening bonus, with lunch/late penalties) and picks the best one.
- Creates the event in the Notion Calendar and marks the Inbox item as `Parsed = true`.
- Sends a **Telegram** message with the scheduled time + top 3 alternatives.

### 2) Daily Plan (07:30 Europe/Rome)
- Pulls today’s events and pending planned tasks.
- Sends a Telegram daily plan message (agenda + top priorities).
- If total Deep Work scheduled is below 90 minutes, it tries to auto-create a **90-min “Focus Block”** today and notifies you on Telegram.

### 3) Evening Reset (21:30 Europe/Rome)
- Pulls today’s planned events.
- For flexible items, attempts to reschedule into the next ~3 days (or earlier if a `Deadline` exists).
- Fixed items can be marked as skipped.
- Sends an evening summary via Telegram.

## Requirements
- n8n (self-hosted or cloud)
- Notion Internal Integration + shared access to the databases
- Ollama running locally (default host: `http://127.0.0.1:11434`)
- Telegram bot token + your chat_id (workflow currently uses a fixed chat_id)

## Notion databases

### Inbox DB (minimum)
- **Name** (Title)
- **Parsed** (Checkbox)

### Calendar DB (expected properties)
- **Name** (Title)
- **When** (Date, with end time)
- **Type** (Select)
- **Flex** (Select: Fixed/Flexible)
- **DurationMin** (Number)
- **Priority** (Number)
- **BufferBefore** (Number)
- **BufferAfter** (Number)
- **Status** (see “Known issues” below)
- **SourceId** (Rich text)
- (Optional) **Deadline** (Date) — used by the reschedule logic

## Setup notes (important)
- Some nodes use workflow variables (e.g. `CAL_DB_ID`), while other nodes still contain hardcoded database IDs.
  Make sure you replace ALL occurrences in:
  - Inbox Trigger database URL
  - Calendar queries in Daily Plan / Focus Block nodes
  - Any “Check Idempotency” / “Create Focus Block” node referencing a databaseId

## Known issues / mismatches to fix (recommended)
- **duration vs durationMin**: Ollama outputs `duration`, but the “Create Calendar Event” node currently maps `durationMin`.
  → Fix by mapping `DurationMin` to `parsed.duration` or changing the parser schema.
- **Status property type mismatch**: some nodes write `Status` as **select**, others as **status**.
  → Standardize the property type in Notion and update the node mappings accordingly.
- **Evening Reset filter uses `Date`** in one query: ensure your DB has that property or switch it to `When`.

## Troubleshooting
### Ollama port already in use
`listen tcp 127.0.0.1:11434: bind...` means something is already using port 11434.
Stop the existing Ollama process or run Ollama on a different port.

---

