# Dashboard Architecture

This document describes the **architecture and design principles** of the
WXPN countdown dashboard engine used across this repository.

It is a **technical reference** explaining how the system works and why it is
structured the way it is. It is not a setup guide or user manual.

---

## Conceptual Model

Each dashboard is a **client-side, state-driven application** where:

- a small set of centralized state values drives the UI
- all panels derive their display from shared state
- charts are computed once and reused
- embedded media is strictly user-initiated

The architecture prioritizes:

- predictable behavior
- mobile usability
- performance on large datasets (≈900–2,000 rows)
- protection against embedded media rate limits
- ease of reuse across multiple countdown variants

---

## High-Level Data & UI Flow

```mermaid
flowchart TB
  %% Bootstrap
  A[User opens index.html] --> B[Load JSON data]
  B --> C[Normalize rows]
  C --> D[Compute ModeSpec]
  D --> E[Precompute chart metrics]

  %% Initial render
  E --> F[Initialize slider + panels]
  F --> G[Render initial state]

  %% Interaction loop
  G --> H[User moves slider]
  G --> I[User clicks chart bar (optional)]
  G --> J[User clicks Last 10 item]

  %% State updates
  H --> K[Update current rank]
  J --> K
  I --> L[Update filter state]

  %% Panel updates
  K --> M[Update Current Song panel]
  K --> N[Update Last 10 Songs panel]
  K --> O[Highlight charts only]

  L --> P[Update Song List panel]

  %% Player lifecycle
  M --> Q[User presses Play]
  Q --> R[Create embedded player iframe]
  R --> S[Playback continues until closed]
```

> Mermaid syntax above is compatible with GitHub’s renderer.

---

## Core Design Decisions

The following design decisions are intentional and form the foundation of the
dashboard engine.

### 1. Centralized State

- The **current rank** is the primary navigation state
- Optional **filter state** augments, but does not replace, rank navigation
- Panels do not manage their own independent state

This prevents divergence between UI regions.

---

### 2. Mode Specification (ModeSpec)

All mode-dependent behavior is centralized in:

```
getModeSpec(APP_CONFIG)
```

The ModeSpec defines:

- active metrics
- visible panels
- panel titles
- song list column structure
- filter title templates

The rest of the engine consumes the ModeSpec without branching on mode.
This avoids scattered conditional logic.

---

### 3. Stable Panels (No Repurposing)

Each panel has a **semantic identity**, for example:

- `panel-artists`
- `panel-genres`
- `panel-cover-artists`
- `panel-current-song`
- `panel-song-list`

Panels are **shown or hidden** by mode.
They are never repurposed to display unrelated content.

This makes layout behavior predictable and testable.

---

### 4. Precomputed Metrics

- Chart metrics are computed **once**, after normalization
- Slider movement does not recompute charts
- Charts only update highlight state during interaction

This ensures smooth interaction even on large datasets.

---

### 5. User-Initiated Media Playback

Embedded players follow strict rules:

- iframes are created only when the user presses Play
- slider movement does not reload players
- playback continues until explicitly closed or replaced

This prevents:
- excessive iframe churn
- unexpected audio playback
- streaming provider rate-limit errors (e.g. HTTP 429)

---

### 6. Mobile-First Interaction Constraints

Mobile behavior is intentional:

- phone layouts default to 1–2 columns
- only essential panels are open by default
- interactive chart filtering is **disabled** on phone layouts

This reduces accidental interaction and improves usability on small screens.

---

## Data Normalization

All incoming rows are processed by a single normalization step before use.

Normalization guarantees:

- consistent UTF-8 string handling
- removal of legacy escape artifacts (e.g. `\'`)
- robust year parsing (string or number)
- consistent decade derivation
- stable string keys for aggregation
- `genres[]` treated as canonical

After normalization, downstream code operates only on normalized fields.

This prevents subtle bugs such as:
- empty charts due to mixed key types
- inconsistent year buckets
- escaped text appearing in labels or tables

---

## Charts & Metrics

Charts are rendered using **D3.js** as SVG horizontal bar charts.

Characteristics:

- responsive to available width
- proportional allocation of label vs plot area
- truncated labels with full-value tooltips
- stable ordering based on frequency

Charts are defined declaratively via metric definitions supplied by the ModeSpec.

---

## Interactive Chart Filtering

Some charts support **click-to-filter** interactions.

Rules:

- clicking a bar filters the Song List panel
- the slider position does **not** change
- filters are cleared via Clear button or Esc key
- filtering is enabled only in 3-column layouts

Tooltips communicate affordance:

- bars: “Click to filter by <value>”
- labels: full value for readability

Which charts are filterable is controlled by configuration.

---

## Song List Panel Behavior

The Song List panel has two distinct modes:

1. **Default mode**
   - shows the last N songs relative to the current rank

2. **Filtered mode**
   - shows all rows matching the active filter
   - remains navigable
   - does not affect slider state

Column structure is determined by the ModeSpec:
- standard mode omits original metadata
- cover mode includes original artist and year

---

## Testing & Stability

The architecture is designed to be testable despite being a single-file app.

Automated tests validate:

- mode-specific behavior
- filtering rules
- normalization guarantees
- visual stability via screenshots

Tests focus on **engine contracts**, not specific countdown content.

---

## What This Architecture Is (and Isn’t)

**This architecture is:**
- static and client-side
- configuration-driven
- optimized for exploration and readability
- resilient to imperfect historical data

**It is not:**
- a framework
- a reactive UI system
- a real-time or streaming visualization engine

---

## Status

This architecture represents a **stable, refactored baseline**.

Future changes should:

- preserve centralized state and ModeSpec
- avoid panel repurposing
- maintain static-host compatibility
- introduce new behavior through configuration first
