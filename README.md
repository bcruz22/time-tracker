[![Time /T](logo.png)](https://bcruz22.github.io/time-tracker/)

# Time /T

*Track local. Save local.*

A small, on-device time tracker for billable work. Runs as an installable web app on desktop browsers, iOS, and iPadOS. No accounts, no servers, no analytics.

**Live app:** [bcruz22.github.io/time-tracker](https://bcruz22.github.io/time-tracker/)

## What it does

- Track billable time per ticket with multiple concurrent stopwatches (up to 10 at once) for real-world context switching
- Both entry modes: live timer with Start / Pause / Stop, or manually edit the start and end times after the fact
- Grouped log view by Ticket # then day, so multiple sessions on the same ticket roll up into one billing entry with a shared total
- One-click **Copy notes** on any session or entire ticket group, notes only, no timestamps or metadata, ready to paste into any ticketing or billing system
- Client Name autocomplete builds up as you use the app
- Status per ticket (Open, Waiting on Customer, Waiting on Internal, Closed). Closed tickets collapse to the bottom of the log
- Today / This Week / This Month running totals at the top, always visible
- Idle warning after configurable inactivity (default 60 min) with Continue / Pause / Discard actions, then a persistent amber dot on the timer card if dismissed
- End of day helper: "Stop all timers" appears when 2 or more are running
- Crash recovery: if a timer was still running when the browser closed, on next open choose to continue from now, cap at last active time, or discard
- Export as CSV, plaintext JSON, or password-encrypted JSON; import any of these back later
- Follows system light/dark preference, with a manual override
- Compact mode: tighter layout, auto-enables under 640px, or force it on/off from the header
- Works offline once installed

## Privacy

All data lives in your device's `localStorage`. Nothing is sent to any server. If you install the app on multiple devices, each device keeps its own private copy. There's no sync.

If you share the app link with someone else, they get their own private, empty instance to fill in. Nothing you enter and nothing they enter ever crosses paths.

If you use the **Export encrypted** feature, the backup file uses AES-GCM with a PBKDF2-derived key (250,000 iterations, SHA-256). If you lose the password, the backup cannot be recovered.

## Install on iPhone / iPad

1. Open [the live app](https://bcruz22.github.io/time-tracker/) in Safari
2. Tap the Share button
3. Tap **Add to Home Screen**
4. Launch it from the icon on your home screen for the full-screen experience

Add to Home Screen also protects your data from Safari's periodic storage cleanup. Bookmarked sites can have their storage cleared after about 7 days of inactivity, but installed sites don't.

## Install on Desktop

1. Open [the live app](https://bcruz22.github.io/time-tracker/) in Chrome, Edge, or another Chromium browser
2. Click the install icon in the address bar (or use browser menu > Install)
3. Or just bookmark and use in the browser. Installation is optional

## Backup workflow

The recommended cadence:

- **Weekly:** export a plain JSON backup and save it somewhere durable (iCloud, OneDrive, USB drive, whatever). If your browser cache gets wiped, this is your safety net.
- **Monthly:** once you've entered the month's time into your billing system, export an encrypted JSON as your archive.

## Files

- `index.html` — the entire application (HTML, CSS, JS in one file)
- `sw.js` — service worker for offline support
- `manifest.json` — PWA metadata
- `icon-180.png`, `icon-192.png`, `icon-512.png` — home screen icons
- `SPEC.md` — design and behavior specification

## Deploying a new version

1. Edit files as needed
2. In `sw.js`, bump `CACHE_NAME` (e.g. `time-tracker-v1` → `v2`) so existing installs pick up the new version
3. Bump `APP_VERSION` in `index.html`
4. Commit and push to `main`; GitHub Pages rebuilds automatically
5. Installed devices will show a "New version available" banner on their next visit

## License

MIT — see [LICENSE](LICENSE).
