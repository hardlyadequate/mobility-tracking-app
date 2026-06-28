# Movement Tracker

A minimal, mobile-friendly weekly fitness tracker built as a single HTML file. No frameworks, no backend, no account required — just open it in a browser and start logging.

---

## What it tracks

The app is built around three movement blocks:

**1. Yoga Standing Postures** (3–4×/week)
Eight poses held for 30 seconds each side, focused on knee health and hip stability. Includes Warrior 1 & 2, Triangle, Revolved Triangle, Warrior 3, Chair, High Lunge, and Horse.

**2. 90/90 Drills** (5–7×/week)
A five-step hip mobility sequence: passive stretching, transitions, external/internal rotation, push-ups, and lateral reach.

**3. Quadrupedal Movement** (5–7×/week)
Four crawling movements performed slowly for one minute each — Bear, Monkey, Frogger, and Crab — targeting body control, shoulder stability, and wrist strength.

---

## Features

- **Daily check-off** — tap any movement to mark it done; tap again to uncheck
- **Movement popups** — tap the `i` button on any movement for a full description, coaching cues, a key tip, and a link to the full guide
- **Weekly block summary** — shows a dot for each day of the week per block, so you can see at a glance how consistently you're hitting each one
- **Day navigation** — switch between days using the day row at the top; dots appear under days with logged activity
- **Week navigation** — arrow buttons let you flick back through previous weeks
- **Summary cards** — shows items completed today, total items, and days active this week
- **Export / Import** — download your full history as a `.json` file and re-import it on any device, so your data isn't locked to one browser

---

## How to use it on your phone

### Option A — Local file (quickest, works offline)
1. Download `index.html` to your computer
2. AirDrop it to your iPhone (or transfer via iCloud/Google Drive/email)
3. Open it in Safari
4. Tap the Share button → **Add to Home Screen**

It will sit on your home screen like an app and works fully offline.

### Option B — GitHub Pages (accessible from any device)
1. Create a free account at [github.com](https://github.com)
2. Create a new repository and upload `index.html` via **Add file → Upload files**
3. Go to **Settings → Pages → Branch → main → Save**
4. After ~60 seconds you'll get a live URL: `https://yourusername.github.io/repo-name`
5. Open that URL on your phone in Safari → **Add to Home Screen**

### Option C — Instant hosting (no account needed)
Drag the file onto [tiiny.host](https://tiiny.host) and get a live URL in seconds.

---

## Backing up your data

The tracker saves to your browser's `localStorage`. This means data is tied to the browser on that specific device — clearing browser data will wipe it.

**To back up:** tap **↓ Export history** — this downloads a `.json` file with all your logged sessions. Save it to iCloud, Google Drive, or email it to yourself.

**To restore:** tap **↑ Import history** and pick the `.json` file. Your history merges back in instantly.

Do this before switching phones or reinstalling your browser.

---

## How the code works

The entire app is a single `.html` file with no external dependencies beyond a Google Fonts import. Everything — structure, styles, and logic — lives in one file.

### Data model

All exercise data is defined as plain JavaScript arrays at the top of the script:

```js
const yogaPoses  = [ { id, name, link, desc, cues, tip }, ... ]
const hipDrills  = [ { id, name, detail, desc, cues, tip }, ... ]
const quadMoves  = [ { id, name, link, desc, cues, tip }, ... ]
```

### State

The app's state is a plain object keyed by date string:

```js
state = {
  "2025-06-23": {
    yoga: Set(["warrior1", "triangle"]),
    hip:  Set(["passive90", "transitions"]),
    quad: Set([])
  },
  ...
}
```

Each category stores a `Set` of completed item IDs for that day. On every change, state is serialised to `localStorage` via `JSON.stringify` (Sets are converted to arrays first).

### Rendering

The app uses a simple manual render cycle — no virtual DOM or reactive framework. A top-level `render()` function calls each sub-renderer in order:

```
render()
  ├── renderWeekLabel()
  ├── renderDays()
  ├── renderWeekSummary()
  ├── renderYoga()
  ├── renderHip()
  ├── renderQuad()
  └── renderSummary()
```

Each renderer reads from `state` and rebuilds its section of the DOM from scratch. This is simple and fast enough for a tracker of this size.

### Weekly summary

`renderWeekSummary()` iterates over all 7 days in the current week and checks each day's state for each block. A day dot is styled as:
- **Empty** — nothing logged
- **Partial** — some items done (shows the count)
- **Full** — all items completed (shows a ✓)

### Popups

Clicking the `i` button calls `showPopup(type, item)`, which injects the item's description, cues, tip, and optional link into a pre-existing overlay `div` and makes it visible. Clicking outside the popup, pressing Escape, or clicking ✕ calls `closePopup()`.

### Export / Import

`exportData()` serialises the full state to JSON, wraps it in a `Blob`, and triggers a download via a temporary `<a>` element.

`importData()` reads the selected file with `FileReader`, parses the JSON, and merges it into the current state — imported data wins on any date conflict. State is then saved and `render()` is called.

---

## File structure

```
index.html
  ├── <style>        — all CSS, scoped with CSS custom properties
  ├── <body>         — header, nav, day row, summary cards, three sections, popup overlay
  └── <script>
        ├── Data     — yogaPoses, hipDrills, quadMoves arrays
        ├── State    — state object, saveState(), loadState()
        ├── Render   — render() and all sub-renderers
        ├── Actions  — toggleItem(), toggleSection()
        ├── Popup    — showPopup(), closePopup()
        ├── Export   — exportData(), importData(), showToast()
        └── Init     — loadState(), render()
```
