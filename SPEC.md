# Time /T — Specification (v0.1.0)

*Track local. Save local.*

A single-user, on-device time tracker for billable work. Runs as an installable web app. No accounts, no servers, no sync. Sibling project to `bcruz22/expenses-tracker`.

**Repo:** `bcruz22/time-tracker`
**Live app (planned):** `bcruz22.github.io/time-tracker`
**License:** MIT

---

## 1. Purpose

Help individuals log time against tickets so they can bill accurately at end of month. Concurrent timers for real-world context switching. Grouped view by ticket so multiple sessions roll up into one billing entry. Notes-only clipboard copy for pasting into external ticketing systems.

## 2. Architecture

- Single-file `index.html` with inline CSS and JS (matches expenses tracker)
- `sw.js` service worker for offline
- `manifest.json` for installability
- `localStorage` for all persistence
- No dependencies, no build step
- GitHub Pages hosted
- `CACHE_NAME` bump on each release to trigger the "New version available" banner

## 3. Data Model

### Entry

Each ticket session is one entry.

```
{
  id: string (uuid),
  ticketNumber: string (required),
  clientName: string (required),
  status: "Open" | "Waiting on Customer" | "Waiting on Internal" | "Closed",
  notes: string,
  startTime: ISO timestamp,
  endTime: ISO timestamp | null (null = still running),
  durationSeconds: number (derived on stop, cached),
  createdAt: ISO timestamp,
  updatedAt: ISO timestamp
}
```

### Active Timer

Running timer state, persisted on every tick so crash recovery works.

```
{
  id: string (uuid, matches entry id once created),
  entryId: string | null (null until first Stop),
  ticketNumber: string,
  clientName: string,
  status: string,
  notes: string,
  startTimestamp: ISO timestamp,
  lastHeartbeat: ISO timestamp,
  state: "running" | "paused" | "completed",
  pausedAt: ISO timestamp | null,
  accumulatedSeconds: number (survives pause)
}
```

### Settings

```
{
  theme: "light" | "dark" | "system",
  compactMode: "auto" | "on" | "off",
  idleThresholdMinutes: number (default 60),
  copyTemplate: "notes-only" (fixed for v0.1.0),
  autocompleteClients: string[] (unique client names ever used),
  lastVersion: string (for update banner)
}
```

## 4. Screens

Everything lives on one page. Three sections stacked:

1. **Header bar:** app name, totals (Today / This Week / This Month), theme toggle, compact toggle, settings icon
2. **Active Timers area:** 1 to 10 timer cards, + button to add, "Stop all" button when 2+ running
3. **Entry Log:** grouped by Ticket #, then day. Closed groups collapse to bottom.

Settings is a modal overlay, not a separate route.

## 5. Timer Card

### Empty state (first card on fresh install)

```
[ Client Name          ] (autocomplete)
[ Ticket #             ]
[ Status: Open       ▼ ]
[ Notes...             ]
                        00:00
                       [ Start ]  [ x ]
```

Fields inline on the card, not below it (departure from the multi-stopwatch reference).

### Running state

- Clock ticks upward in HH:MM:SS
- Start button becomes Pause
- Stop button appears
- Fields remain editable while running
- Amber dot appears if idle warning was dismissed

### Paused state

- Clock frozen
- Pause becomes Resume
- Stop still available

### Completed state (after Stop)

- Entry is saved to log
- Clock shows final duration
- Buttons become **Start Again** and **Clear**
  - **Start Again:** creates a new entry, Client + Ticket # + Status pre-filled, Notes clears, timer resets to 00:00 and runs
  - **Clear:** wipes all fields, timer to 00:00, card returns to empty state
- X button removes the card entirely

### Card add / remove

- Fresh install: 1 empty card visible
- + button in Active Timers area adds one (max 10)
- X removes; if timer is running, confirm

### Stop with empty required fields

If Client Name or Ticket # is blank when Stop is hit: modal asks to fill them or discard the session.

## 6. Concurrent Timers

- All timers can run simultaneously
- No auto-pause when starting another
- Tech is responsible for what they actually count
- "Stop all timers" button appears in the Active Timers area when 2 or more are running

## 7. Entry Lifecycle

- Timer creates an entry on first Stop
- Running timer state is separate from entry until stopped
- Subsequent Stop / Start Again on the same card creates new entries with same Client + Ticket #
- Entries roll up in the log view under the shared Ticket # group

## 8. Entry Log Display

### Grouping

Two-level hierarchy:

```
[Ticket #12345] — Bank of Example — 2h 15m total — Open ▼
    Oct 15
        09:30–10:15 (45m) — "Investigated login failures, reset MFA"
        14:00–15:15 (1h 15m) — "Root cause identified, patched module"
    Oct 16
        11:00–11:15 (15m) — "Verified fix with user"

[Ticket #12346] — Credit Union Example — 30m total — Waiting on Customer ▼
    ...

[Closed] ▶ (collapsed by default)
    [Ticket #12340] — ... — 1h total ▶
    [Ticket #12339] — ... — 45m total ▶
```

### Sort

- Non-Closed groups: most recently updated at top
- Closed section: at bottom, collapsed by default
- Within a ticket group: chronological, most recent day first

### Interactions per group

- Ticket header shows total time, most recent status
- Expand/collapse the group
- **Copy notes** button on the group: concatenates all sessions' notes newline-separated, no timestamps
- **Copy notes** button on each session: that session's notes only

### Interactions per session (entry)

- Tap to edit inline: Client Name, Ticket #, Status, Notes, Start Time, End Time
- Duration recalculates from time edits
- Delete button (confirms)
- All fields editable after save

### Ticket # autocomplete

- Only surfaces when adding a new entry that could join an existing group
- On the timer card, Ticket # is free-typed
- If the typed value matches an existing ticket, the group absorbs the new session on Stop
- No dropdown suggestions for entirely new tickets

### Client Name autocomplete

- Type-ahead from `autocompleteClients` on every entry
- New values append to the list on save

## 9. Copy to Clipboard

Format: **notes only, no timestamps, no metadata.**

Per session:
```
Investigated login failures, reset MFA
```

Per ticket group (multiple sessions):
```
Investigated login failures, reset MFA

Root cause identified, patched module

Verified fix with user
```

Blank line between session notes for readability in the destination system.

## 10. Duration Display

Exact time throughout. No 15-minute rounding logic in v0.1.0.

- Running timers: `HH:MM:SS`
- Saved entries: `Xh Ym` (e.g., "1h 15m", "45m", "8m")
- Group totals: `Xh Ym`
- Header totals: `Xh Ym`

## 11. Header Totals

Three running totals, updated live:

- **Today:** all entry time from midnight local
- **This Week:** current week starting Monday (configurable in settings)
- **This Month:** current calendar month

All statuses count, including Closed.

## 12. Idle Warning

- Trigger: 60 minutes continuous run without pause or stop (configurable in settings)
- First action: toast with three inline buttons — **Continue** / **Pause** / **Discard**
- Dismissed or ignored: amber dot appears on the timer card as a persistent visual reminder, no further toasts
- Amber dot clears on next user action (Pause, Stop, Start Again, or manual dismiss)

## 13. End of Day Helper

- "Stop all timers" button appears in Active Timers area when 2 or more timers are running
- Single click stops all running timers
- If any have empty required fields, prompts once with all incomplete entries listed
- Completed cards enter the standard Start Again / Clear state

## 14. Crash Recovery

On app load, if any active timer has `state: "running"` and `lastHeartbeat` older than 30 minutes:

Modal:
```
Timer for [Client] / [Ticket #] was still running.
Last active [X hours Y minutes ago] at [HH:MM AM/PM].

[ Continue counting from now ]
[ Cap at last active time (X:XX) ]
[ Discard this session ]
```

Heartbeat writes to localStorage every 10 seconds while a timer is running.

## 15. Settings Modal

- **Theme:** Light / Dark / System (matches expense tracker)
- **Compact mode:** Auto / On / Off (matches expense tracker: Auto enables under 640px)
- **Idle warning threshold:** minutes input, default 60
- **Manage autocomplete history:** view / delete client names
- **Data management:**
  - Export CSV
  - Export JSON (plaintext)
  - Export JSON (encrypted) — password prompt with strength requirements matching expense tracker
  - Import (auto-detects format)
  - Clear all data (confirm twice)
- **About:** version, link to repo

## 16. Import / Export

Same crypto pattern as expenses tracker:

- Plain JSON: full data model dump
- Encrypted JSON: AES-GCM, PBKDF2 with 250,000 iterations, SHA-256
- CSV: flat entry list, columns: Ticket #, Client Name, Status, Start Time, End Time, Duration, Notes
- Password strength requirements dialogue reused from expense tracker
- Import merges by entry `id` (duplicates skipped, updates by `updatedAt` timestamp)

Export scope: all entries + settings + autocomplete history. Active timers not exported (they're transient state).

## 17. Status Behavior

- Default new entry status: **Open**
- Statuses **Open**, **Waiting on Customer**, **Waiting on Internal** display in main log, expanded by default
- Status **Closed** rolls into a "Closed" section at bottom, collapsed by default
- Status change on any session of a ticket updates the group's displayed status to the most recent session's status
- Closed entries still count toward Today/Week/Month totals

## 18. Compact Mode

Matches expense tracker pattern:

- Auto: enables under 640px viewport
- On/Off: manual override in header and settings
- When compact: session detail lines behind tap-to-expand, tighter spacing throughout, timer cards shrink vertically

## 19. Theme

- System-follow by default
- Manual override in header (icon toggle) and settings
- Same CSS variable-based light/dark palette as expenses tracker

## 20. PWA / Offline

- `manifest.json` with icons (180, 192, 512)
- `sw.js` caches shell, works offline after first load
- Add to Home Screen supported on iOS/iPadOS/desktop
- "New version available" banner via `CACHE_NAME` bump on each release

## 21. MVP Scope (v0.1.0)

Everything above in sections 1–20.

## 22. Later Scope (not v0.1.0)

- 15-minute rounding options
- User-configurable copy template with tokens
- Ticket filtering / search bar
- Date range filtering
- Weekly export reminder
- Ticket # autocomplete dropdown for entirely new entries
- Status change history / audit
- Timer card reordering
- Multiple tabs of the app open at once (currently: last write wins)

## 23. Naming and Branding

- **App name:** Time /T
- **Tagline:** Track local. Save local.
- **Repo:** `bcruz22/time-tracker`
- **URL:** `bcruz22.github.io/time-tracker`
- Icon: to be designed (terminal / clock hybrid, matching expense tracker's visual weight)

## 24. Release / Versioning

- Semantic versioning: MAJOR.MINOR.PATCH
- v0.1.0 = MVP as specified
- Release process matches expense tracker: bump `CACHE_NAME` in `sw.js`, commit, push to `main`, GitHub Pages rebuilds
- README follows expense tracker's structure


