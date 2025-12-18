# Dashboard Engine Overview

This repository contains a set of **single-page, self-contained dashboard apps**
used to explore various **WXPN music countdowns**.

Each countdown lives in its own subdirectory and uses the same underlying
dashboard engine, configured via a small JavaScript object (`APP_CONFIG`)
inside each page.

This document describes the **dashboard engine itself** — its behavior,
assumptions, and design decisions — rather than any one specific countdown.

---

## What the Dashboard Is Designed For

The dashboard engine is designed to be:

- fast and responsive
- readable on desktop, tablet, and phone
- safe for embedded media rate limits
- easy to reuse for future countdown variants
- deployable as a single static HTML file

Each dashboard:
- runs on GitHub Pages
- runs on any simple static web server
- does not require a build step or external framework

---

## Live Usage & Local Development

Each dashboard is a **single HTML file**.

To run locally, you must use a local web server (not `file://`):

```bash
python3 -m http.server
```

Then open the relevant countdown directory, for example:

```
http://localhost:8000/greatest-cover-songs/
```

---

## Core Interaction Model

### Countdown Navigation
- A slider moves through the countdown from higher ranks toward #1
- The default direction emphasizes discovery, starting with lesser-known items
- The **“Last 10”** panel allows quick jumping to nearby ranks

### Current Item Panel
- Displays album art (dataset-provided only)
- Shows title, artist, and year
- For cover countdowns: original artist and original year
- External links (Spotify / Deezer / Bandcamp) when available

### Embedded Player (Rate-Limit Safe)
To avoid media provider throttling (429 errors):

- The embedded player iframe is **created only when Play is pressed**
- The player does **not reload** when the slider changes
- Playback continues until the user explicitly presses Play again
- Closing the player removes the iframe entirely

This design allows fast browsing without triggering provider limits.

---

## Charts & Metrics

The dashboard renders **SVG-based horizontal bar charts**
using plain JavaScript (no charting libraries).

Common metric types include:
- artists
- albums
- years
- decades
- genres

Cover-specific countdowns may additionally include:
- original artists
- original years
- original decades
- original songs

Charts are:
- responsive to available width
- readable in one-column mobile layouts
- label-aware (space allocated dynamically for text)

Which charts appear — and in what order — is controlled by configuration,
not hard-coded DOM structure.

---

## Responsive Layout

The dashboard layout adapts automatically:

- Desktop: 3 columns
- Tablet: 2–3 columns
- Phone portrait: 1 column

On phones:
- **Current Song** and **Last 10** panels are open by default
- All other panels start collapsed but can be expanded

---

## Data Expectations

Each dashboard expects a JSON array of objects.

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
  "genres": ["classic rock", "psychedelic rock"],
  "media": {
    "spotify": { "embed_src": "…" },
    "deezer": { "embed_src": "…" }
  }
}
```

### Normalization handled by the dashboard
- trims stray whitespace and CR/LF
- unescapes PHP-style strings (`You\'ve` → `You've`)
- parses years robustly (strings or numbers)
- derives decades consistently
- treats `genres[]` as the canonical genre source
- ensures chart keys are string-stable (prevents empty charts)

Legacy fields are supported only as fallback for older datasets.
New datasets should use canonical fields (for example, `genres[]`) and avoid PHP-serialized values.

---

## Configuration & Reuse

Dashboard behavior is controlled via a small configuration object inside each countdown’s `index.html`.

Each countdown page defines its own configuration and data source, allowing multiple dashboards to coexist within a single repository.

Examples of configurable behavior:
- countdown name
- data file path
- cover vs standard mode
- which chart metrics appear and in what order

This makes the engine reusable for:
- non-cover countdowns
- alternate rankings
- countdowns with fewer or different metrics

---

## What This Is (and Isn’t)

**This is:**
- a purpose-built dashboard engine
- optimized for static hosting
- designed for clarity and exploration

**This is not:**
- a framework
- a React / Vue / D3 application
- a streaming or real-time visualization system

---

## Status

- ✅ Stable baseline
- 🚀 Ready for public use and future adaptations
