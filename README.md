# WXPN Countdown Dashboards

This repository contains a collection of **interactive, single-page dashboards**
for exploring various **WXPN music countdowns**.

The project began as a way to visualize trends in the
*WXPN 885 Greatest Cover Songs* countdown, and has since expanded to support
multiple countdowns, each hosted as its own dashboard under a shared structure.

These dashboards are designed to be:
- fast
- readable on a broad range of devices
- safe for embedded media rate limits
- easy to reuse for future countdown variants

---

## Live Usage

Dashboards are published via **GitHub Pages** and accessed from the landing page
for this repository.

```
https://ericknappe.github.io/countdowns
```

Each countdown lives in its own subdirectory and is implemented as a
**single, self-contained HTML file**.

Dashboards can be run:
- directly from GitHub Pages
- from any simple static web server
- locally (using a local web server, not `file://`)

Example local run:

```bash
python3 -m http.server
```

Then open:

```
http://localhost:8000/index.html
```

---

## Features

### Countdown Navigation
- A slider moves through the countdown from higher ranks toward **#1**
- Designed to start with lesser-known entries and progress toward the top
- A **“Last 10 Songs”** panel allows quick navigation toward the end of the countdown

### Current Song Panel
- Album art (dataset-provided only)
- Song title and artist
- Year information
- For cover countdowns: original artist and original year
- External links (Spotify / Deezer / Apple Music / Deezer when available)

### Embedded Player (Rate-Limit Safe)
- Player iframe is created only when **Play** is pressed
- Player does not reload when the slider moves
- Playback continues until the user explicitly presses Play again
- Prevents Spotify / Deezer 429 errors during fast scrolling

### Charts & Metrics
SVG-based horizontal bar charts rendered with plain JavaScript  
(no external charting libraries).

Standard countdowns typically include:
- Artists
- Albums
- Years
- Decades
- Genres

Cover countdowns may also include:
- Most covered original artists
- Most frequent cover artists
- Original years / decades
- Cover years
- Most covered original songs

Charts:
- adapt to screen size
- remain readable in one-column mobile layouts
- allocate label space proportionally for clarity

### Interactive Chart-Click Filtering (Desktop/Tablet Only)
On supported dashboards, **click a bar (or label)** in eligible charts to filter the song list panel.

- **Enabled only when the layout is 3 columns** (desktop/tablet).  
  Disabled on phone layouts (including phone landscape in 1–2 column mode).
- Clicking a chart value sets an *active filter* and populates the song list panel with **all matches** (no “last 10” limit).
- The slider does **not** jump when a filter is applied.
- Clear the filter via the **Clear** button or **Esc**.

Filtering is configured per countdown via:

```js
APP_CONFIG.interactiveFilters = {
  enabled: true,
  filterableCharts: ["genres","origArtists","coverArtists","origYears"] // cover mode example
};
```

Supported filter types by mode:

**Cover mode**
- `genres` → genre
- `origArtists` → original artist
- `coverArtists` → cover artist
- `origYears` → original year

**Standard mode**
- `genres` → genre
- `artists` → artist
- `years` → year

Charts that are not present for a given countdown are ignored.

### Responsive Layout
- Desktop: 3 columns
- Tablet: 2–3 columns
- Phone portrait: 1 column, landscape: 2 columns

On phones:
- Only **Current Song** and **Last 10 Songs** panels are open by default
- All other panels start collapsed

---

## Data Format

Dashboards expect a JSON array of song objects.

Minimal example:

```json
{
  "id": "19621",
  "rank": 1,
  "artist": "Jimi Hendrix",
  "song": "All Along The Watchtower",
  "album": "Electric Ladyland",
  "releaseDate": "1968",
  "albumArt": "https://…",
  "originalArtist": "Bob Dylan",
  "originalYear": "1967",
  "genres": ["psychedelic rock", "classic rock", …],
  "media": {
    "spotify": { "embed_src": "…" },
    "deezer": { "embed_src": "…" },
    "applemusic": { "previewUrl": "…", "url": "…" }
  }
}
```

Legacy datasets may include PHP-serialized genre fields; these are supported as fallback.

### Normalization handled by the app
- trims stray whitespace and CR/LF
- unescapes legacy escaped text (e.g., `You\'ve` → `You've`)
- parses years robustly (strings or numbers)
- derives decades consistently
- treats `genres[]` as the canonical genre source
- ensures chart keys are string-stable (prevents empty charts)

---

## Reuse & Future Variants

Dashboard behavior is controlled via a small configuration object inside each
countdown’s `index.html` (e.g., `APP_CONFIG` and `APP_CONFIG.mode`), rather than
hard-coded DOM assumptions.

---

## Testing (Optional)

Automated UI tests use **Playwright**.

Typical commands:

```bash
npm ci
npm test
```

To update screenshots:

```bash
npm run test:update
```

Artifacts (screenshots, traces) can be uploaded in CI (see workflow under `.github/workflows/` if present).

---

## What This Repo Is (and Isn’t)

**It is:**
- a set of polished, static dashboards
- optimized for multiple devices
- designed to avoid common embed pitfalls

**It is not:**
- a framework
- a React/Vue app
- a streaming data visualization system

---

## License & Credits

Data and concept: **WXPN – 885 Greatest Cover Songs**  

This is a fan-made, non-commercial project and is not an official WXPN property.

---

## Status

✅ **Stable baseline**  
🚀 Ready for public use and future adaptations
