# Dashboard Development Guide

This document describes how to **create, adapt, and validate** new countdown
dashboards using the WXPN Countdown Dashboard engine.

It assumes familiarity with:
- HTML / CSS
- JavaScript
- JSON data structures

No build tools or frameworks are required.

---

## Mental Model (Read This First)

A dashboard is a **single self-contained HTML file** driven by:

1. **Configuration** (`APP_CONFIG`)
2. **Mode specification** (`getModeSpec`)
3. **Normalized data**
4. **Stable panel containers**

You do **not** wire charts to DOM cells manually.
You **do not** repurpose panels between modes.
You **do** configure behavior declaratively and let the engine render.

---

## Creating a New Countdown (Checklist)

### 1. Create a new subdirectory

```
countdowns/<new-countdown-slug>/
```

Example:

```
countdowns/greatest-songs-by-women/
```

---

### 2. Copy a known-good baseline dashboard

Copy an existing, working dashboard:

```
countdowns/<existing-slug>/index.html
```

Rename it to:

```
countdowns/<new-countdown-slug>/index.html
```

**Do not start from scratch.**
The baseline includes normalization, filtering, playback rules, and layout logic.

---

### 3. Add your data

```
countdowns/<new-countdown-slug>/data/
  your_data_file.json
```

Dashboards expect a JSON **array** of song objects.

---

### 4. Edit `APP_CONFIG` (the only required edits)

At the top of `index.html`, update:

```js
const APP_CONFIG = {
  countdownName: "WXPN 885 Greatest Cover Songs",
  dataFile: "data/your_data_file.json",
  mode: "cover", // or "standard"
  metrics: [
    "genres",
    "artists",
    "years"
  ]
};
```

Key fields:

| Field | Purpose |
|------|--------|
| `countdownName` | Displayed in the page title and header |
| `dataFile` | Relative path to JSON |
| `mode` | `standard` or `cover` |
| `metrics` | Ordered list of charts to enable |

You should **not** modify chart rendering code to add/remove charts.

---

## Modes and What They Mean

### Standard Mode
Used for countdowns of original recordings.

- Panels include:
  - artists
  - albums
  - years
  - decades
  - genres
- Song list **does not** include original artist columns
- Filtering supports:
  - artist
  - year
  - genre

### Cover Mode
Used for cover-song relationship countdowns.

- Panels may include:
  - original artists
  - cover artists
  - original years / decades
  - cover years
  - most covered original songs
- Song list **includes** original artist and year
- Filtering supports:
  - genre
  - original artist
  - cover artist
  - original year

Mode behavior is defined centrally via `getModeSpec(APP_CONFIG)`.

---

## Adding or Removing Charts

Charts are enabled by **metric keys**, not DOM wiring.

Example:

```js
metrics: [
  "orig_artists",
  "cover_artists",
  "orig_years",
  "genres"
]
```

If a metric is listed:
- its panel is shown
- its chart is rendered
- its ordering is respected

If a metric is omitted:
- the panel is hidden
- no placeholder content is shown

Do **not** reuse panels for different charts.

---

## Interactive Chart Filtering (Optional)

Filtering is configured per countdown:

```js
APP_CONFIG.interactiveFilters = {
  enabled: true,
  filterableCharts: ["genres", "origArtists", "coverArtists"]
};
```

Rules:
- Filtering is **desktop/tablet only**
- Disabled automatically on phone layouts
- Clicking a bar filters the song list
- Slider position does not change
- Clear via:
  - Clear button
  - Esc key

Charts not listed in `filterableCharts` behave normally.

---

## Data Normalization (Important)

All data rows pass through:

```
normalizeRow(row)
```

Normalization handles:
- legacy escaped strings (e.g. `\'`)
- year parsing
- decade derivation
- genre fallbacks
- stable aggregation keys

---

## Panel Layout Rules

Each panel has a **semantic ID**, for example:

```
panel-artists
panel-genres
panel-cover-artists
panel-song-list
panel-current-song
```

Rules:
- Panels are shown or hidden by mode
- Panels are not repurposed
- Grid layout adapts, panel identity does not

Drag-and-drop (if enabled) applies to entire panels, not chart internals.

---

## Local Development

Run a simple static server:

```bash
python3 -m http.server
```

Then open:

```
http://localhost:8000/countdowns/<new-countdown-slug>/
```

Do **not** use `file://`.

---

## Testing (Optional but Recommended)

Playwright tests exist to validate:

- mode behavior
- filtering rules
- normalization guarantees
- visual stability

Typical commands:

```bash
npm ci
npm test
```

Update screenshots:

```bash
npm run test:update
```

Tests validate **engine behavior**, not specific countdown content.

---

## Common Mistakes to Avoid

- ❌ Wiring charts directly to grid cells
- ❌ Adding conditional DOM logic instead of updating ModeSpec
- ❌ Repurposing panels between modes

---

## Philosophy

The dashboard engine is intentionally:
- static
- explicit
- conservative about interaction
- resilient to imperfect historical data

When in doubt:
> **Change configuration first, not code.**
