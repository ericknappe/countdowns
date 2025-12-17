# WXPN 885 Greatest Cover Songs – Countdown Dashboards

This repository contains a **single‑page, self‑contained dashboard app** created for exploring the  
**WXPN 885 Greatest Cover Songs** countdown. Additional dashboards have been added for other **WXPN Countdowns**

The dashboard is designed to be:
- fast
- readable on all devices
- safe for embedded media rate limits
- easy to reuse for future countdown variants

---

## Live Usage

Each app is a **single HTML file** that can run:

- on **GitHub Pages**

```
https://ericknappe.github.io/countdowns
```

- via any simple static server
- locally (with a local web server, not `file://`)

Example local run:

```bash
python3 -m http.server
```

Then open:

```
http://localhost:8000/
```

---

## Features

### Countdown Navigation
- Slider moves through the countdown from **rank NNN → rank 1**
- Start with lesser‑known songs and progress toward the top
- “Next 10 Songs” panel allows instant jumping to nearby ranks

### Current Song Panel
- Album art (dataset‑provided only)
- Song title, artist, original artist (if applicable)
- Original year and cover year
- External links (Spotify / Deezer / Bandcamp when available)

### Embedded Player (Rate‑Limit Safe)
- Player iframe is **created only when Play is pressed**
- Player does **not reload** when the slider moves
- Playback continues until the user explicitly presses Play again
- Prevents Spotify / Deezer 429 errors during fast scrolling

### Charts & Metrics
SVG‑based horizontal bar charts (no external libraries):

- Most Frequent Cover Artists
- Most Covered Original Artists
- Most Covered Original Years
- Most Covered Original Decades
- Most Frequent Cover Years
- Most Covered Original Songs
- Most Represented Genres

Charts:
- adapt to screen size
- allocate label space proportionally
- remain readable in 1‑column mobile layouts

### Responsive Layout
- Desktop: 3 columns
- Tablet: 2–3 columns
- Phone portrait: 1 column

On phones:
- Only **Current Song** and **Next 10 Songs** are open by default
- All other panels start collapsed

---

## Data Format

The dashboard expects a JSON array of song objects.

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
  "genre": "a:4:{i:0;s:16:"psychedelic rock";…}",
  "media": {
    "spotify": { "embed_src": "…" },
    "deezer": { "embed_src": "…" }
  }
}
```

### Normalization handled by the app
- Trims stray whitespace and CR/LF
- Unescapes PHP‑style strings (`You\'ve` → `You've`)
- Robust year parsing (strings or numbers)
- PHP‑serialized genres parsed safely
- All chart keys canonicalized as strings (prevents empty charts)

---

## Reuse & Future Variants

This dashboard is intentionally structured so it can be reused for:

- non‑cover countdowns
- alternate rankings
- fewer or different chart metrics

Metric meaning is **configured in JavaScript**, not hard‑coded into DOM IDs.

---

## What This Repo Is (and Isn’t)

**It is:**
- a set if polished, ready dashboards
- optimized for multiple devices
- designed to avoid common embed pitfalls

**It is not:**
- a framework
- a React/Vue app
- a streaming data visualization system

---

## License & Credits

Data and concept: **WXPN – 885 Greatest Cover Songs**  
Dashboard implementation: custom, purpose‑built

---

## Status

✅ **Stable baseline**  
🔒 Further changes should be intentional and scoped  
🚀 Ready for public use and future adaptations
