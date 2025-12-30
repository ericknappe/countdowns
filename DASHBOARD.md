# Dashboard Engine Overview

This document describes how the **WXPN Countdown Dashboard engine** works.
It is intentionally **conceptual**, not a line‑by‑line code reference.

The goal is to explain:
- how the dashboard is structured
- what guarantees the engine makes
- how different countdown modes are supported
- how interaction and filtering behave

---

## What the Dashboard Is

The dashboard is a **single‑page, single‑file HTML application** designed to visualize
music countdown data (e.g. WXPN’s various “Greatest …” countdowns).

It is designed to be:
- static (no backend required)
- embeddable on GitHub Pages or any static host
- resilient to inconsistent historical datasets
- reusable across multiple countdown variants

The dashboard is **not** a general charting framework and is not intended to be
extended dynamically at runtime.

---

## Libraries Used

The dashboard deliberately minimizes dependencies, but it **does** rely on:

- **D3.js** — used for rendering SVG‑based horizontal bar charts and handling
  scales, axes, and transitions.

No other third‑party charting or UI frameworks are used.
All layout, state management, and interaction logic is written in plain JavaScript
and CSS inside the HTML file.

---

## Engine Architecture

The engine is organized around three core concepts:

### 1. Mode Specification

All mode‑specific behavior is centralized in a single function:

```
getModeSpec(APP_CONFIG)
```

This function returns a **ModeSpec object** that defines:

- which metrics are active
- which panels are visible
- panel titles
- song list column structure
- filter title templates

Supported modes include:
- **standard** (original recordings)
- **cover** (cover relationships)

The rest of the dashboard consumes the ModeSpec and does not need to know
which mode it is operating in.

This avoids scattering `if (mode === ...)` logic throughout the code.

---

### 2. Data Normalization

All incoming rows are normalized once, at load time, by a single function:

```
normalizeRow(row)
```

Normalization guarantees:

- consistent string handling (UTF‑8)
- removal of legacy escape artifacts (e.g. `\'`)
- robust year parsing from strings or numbers
- derived decade values
- stable string keys for chart aggregation
- consistent genre handling (`genres[]` preferred)

After normalization, the rest of the dashboard operates on **normalized fields only**.

This prevents subtle bugs such as:
- empty charts caused by mixed key types
- inconsistent year bucketing
- display of escaped text in labels or tables

---

### 3. Centralized Interaction State

Interactive behavior is handled through centralized state modules rather than
direct DOM coupling.

Key examples:

- **filterState**
  - tracks whether a filter is active
  - defines how rows are matched
  - controls clearing behavior (Clear button / Esc key)

- **player state**
  - ensures embedded players load only on explicit user action
  - prevents reloads during slider movement
  - avoids media provider rate‑limit errors

This approach keeps interactions predictable and testable.

---

## Panels and Layout

Each chart or UI region has a **stable, semantic panel container**:

- `panel-artists`
- `panel-genres`
- `panel-albums`
- `panel-decades`
- `panel-years`
- `panel-cover-artists`
- `panel-cover-years`
- `panel-current-song`
- `panel-song-list`

Panels are **shown or hidden by mode**.
Panels are not repurposed to display unrelated content.

The grid layout adapts to screen size, but panel identity remains stable.

---

## Charts & Metrics

Charts are rendered using **D3.js** as SVG horizontal bar charts.

General properties:
- responsive to available width
- proportional label/plot area allocation
- truncated labels with full‑value tooltips
- stable ordering based on aggregated counts

Charts are declaratively defined via metric definitions in the ModeSpec, including:
- category extraction
- minimum count thresholds
- maximum number of entries
- label and tooltip behavior

---

## Interactive Chart Filtering

Some charts support **click‑to‑filter** interactions.

Behavior:
- Clicking a bar filters the song list panel to matching rows
- The slider position does **not** change
- Filters can be cleared via:
  - the Clear button
  - the Esc key

Filtering is intentionally gated:

- Enabled only when the layout has **3 columns** (desktop / tablet)
- Disabled on phone layouts to prevent accidental activation

Visual affordances:
- bars show tooltips such as:
  - `Click to filter by Bob Dylan`
- labels show value‑only tooltips for readability

Which charts are filterable is controlled per countdown via configuration.

---

## Song List Panel

The song list panel has two modes:

1. **Default**
   - shows the last N songs relative to the current slider position

2. **Filtered**
   - shows all matching rows for the active filter
   - remains navigable
   - does not affect slider position

Column structure is defined by the ModeSpec:
- standard mode omits “Original” columns
- cover mode includes original artist / year

---

## Media Playback Model

Embedded players (Spotify, Deezer, Apple Music previews) follow strict rules:

- players load only on explicit user action
- scrolling or slider movement does not trigger reloads
- playback state is user‑controlled

This design prevents:
- excessive iframe churn
- provider rate‑limit errors
- unexpected audio playback

---

## Testing Philosophy

The dashboard is testable despite being a single‑file app.

Automated tests focus on:
- mode behavior (standard vs cover)
- interactive filtering rules
- normalization guarantees
- visual stability via screenshots

Tests validate **engine contracts**, not specific countdown data.

---

## What This Document Is (and Isn’t)

**This document explains:**
- how the dashboard engine is structured
- what guarantees it provides
- how modes and interactions work

**It does not:**
- document every function
- explain D3 internals
- describe individual countdown content

For usage, deployment, and data examples, see `README.md`.
