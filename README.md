# Personal Calendar Optimizer (n8n + Notion + Ollama)

An n8n automation that takes tasks written in plain English (with constraints like duration, deadlines, preferred time windows, and buffers), analyzes your existing schedule, and automatically time-blocks everything into a Notion Calendar.

✅ **AI parsing is powered by Ollama (local LLM)**  
✅ **Notifications are sent via Telegram only (no email)**

## Why this exists
I kept capturing tasks, but I’d still spend time manually figuring out **when** to do them.  
This workflow converts an “idea inbox” into an **optimized calendar plan**—automatically.

---

## What it does

### 1) Capture & Schedule (Inbox ➜ Calendar)
- You drop a quick request into a Notion **Inbox** database (example below)
- **Ollama** parses constraints (duration, deadline, windows, buffers, preferences)
- It fetches all existing events from your Notion **Calendar** database
- It finds the best free slot and creates the time block in Notion
- It sends you a confirmation + a few fallback alternatives (**Telegram**)

### 2) Daily Plan (every morning)
- Builds today’s agenda from Notion Calendar
- Highlights top priorities
- Auto-creates a **Focus Block** if you’re missing deep work time
- Sends the plan via **Telegram**

### 3) Evening Reset (every night)
- Checks what’s still **Planned** but not **Done**
- Re-schedules **Flexible** tasks before their deadline (or within a configurable horizon)
- Marks **Fixed** tasks as Skipped (or sends them back to Inbox for manual decision)
- Sends a recap via **Telegram**

---

## Example input (what you write in Notion Inbox)

**Plain English**
- “Deep work 90 min before Thursday, morning preferred, 15 min buffer”
- “Gym 45 min tomorrow after 6pm, avoid lunch, buffer 10m”

**Optional structured (if you want)**
```yaml
title: "Deep work"
duration: 90m
deadline: "2026-01-22 18:00"
window:
  start: "2026-01-20 09:00"
  end:   "2026-01-22 18:00"
type: "Deep work"
priority: 4
buffers:
  before: 15
  after: 10
preferences:
  preferMorning: true
  avoidLunch: true
  avoidLate: true
