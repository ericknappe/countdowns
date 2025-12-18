# Dashboard Architecture

This document describes the **architecture and design principles** of the
countdown dashboard engine used across this repository.

It is intended as a **technical reference** explaining how the system works
and why it is structured the way it is. It is not a setup guide or user manual.

---

## Conceptual Model

Each dashboard is a **client-side, state-driven application** where:

- a single primary state value (the current rank) drives the UI
- all panels derive their display from that shared state
- charts are precomputed for performance and stability
- embedded media is strictly user-initiated

The architecture prioritizes:
- predictable behavior
- mobile usability
- performance on large datasets
- protection against embedded media rate limits

---

## Data & UI Flow

```mermaid
flowchart TB
  %% Data & bootstrap
  A[User opens index.html] --> B[Load JSON data]
  B --> C[Normalize & enrich rows]
  C --> D[Precompute chart metrics]

  %% Initial render
  D --> E[Initialize slider bounds]
  E --> F[Render initial UI state]

  %% Interaction loop
  F --> G[User moves slider or clicks Last 10 item]
  G --> H[Update current rank state]

  %% Panel updates
  H --> I[Update Current Song panel]
  H --> J[Update Last 10 Songs panel]
  H --> K[Update charts (highlight only)]

  %% Player lifecycle
  I --> L[User presses Play]
  L --> M[Create embedded player iframe]
  M --> N[Playback continues until closed]

  %% Notes
  K -.-> |Charts are precomputed| D
```

---

## Core Design Decisions

The following decisions shape how the dashboard behaves and are intentional.

1. **Single source of truth**
   - The current rank is the only state that drives panel updates

2. **Charts are precomputed**
   - No chart recomputation occurs during slider interaction
   - Prevents jank and keeps interaction responsive

3. **Embedded media is user-initiated**
   - Player iframes are created only when the user presses Play
   - Prevents rate-limit (429) errors from streaming providers

4. **Player lifecycle is independent**
   - Playback does not restart when the current rank changes
   - The player is replaced only when the user explicitly presses Play again

5. **Panels are slots, not meanings**
   - Panel positions are structural slots
   - The meaning of each panel is defined by configuration, not DOM IDs

6. **Mobile behavior is intentional**
   - On phones, only essential panels are open by default
   - All other panels are accessible but collapsed

---

## Major Components

### Data Normalization
- Trims stray whitespace and line breaks
- Unescapes PHP-style strings
- Parses years robustly (strings or numbers)
- Derives decades consistently
- Treats `genres[]` as the canonical genre source
- Ensures all chart keys are string-stable

Legacy fields are supported only as fallback for older datasets.
PHP-serialized genre fields should not be used in new datasets.

---

### Metric Computation
- Metrics are computed once after normalization
- Each metric produces a ranked frequency list
- Charts render from this cached data

Metrics enabled for a dashboard are controlled by configuration.

---

### Current Song Panel
- Displays metadata for the active rank
- Does not manage playback state
- Updates instantly when the rank changes

---

### Last 10 Songs Panel
- Displays the next set of songs **toward rank #1**
- Acts as a navigation aid rather than a summary
- Clicking an item updates the current rank state

---

### Embedded Player
- Created lazily on user action
- Uses provider-specific embed URLs
- Removed entirely when closed
- Is not affected by slider movement

---

## Configuration Model

Each dashboard defines a small configuration object that controls:

- countdown name
- data file location
- cover vs standard mode
- enabled metrics and their order
- optional rank bounds overrides

This allows multiple dashboards to coexist within a single repository
without shared state or runtime conditionals.

---

## What This Architecture Is (and Isn’t)

**This architecture is:**
- static and client-side
- optimized for exploration and readability
- designed for predictable performance

**It is not:**
- a framework
- a reactive UI system
- a real-time or streaming visualization engine

---

## Status

This architecture represents a **stable baseline**.

Future changes should:
- preserve the core design decisions
- remain compatible with static hosting
- avoid introducing hidden state or implicit behavior
