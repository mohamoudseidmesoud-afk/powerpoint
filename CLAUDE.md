# CLAUDE.md — Fritz Haber Interactive Timeline

## Project Overview

A single-file interactive HTML biography timeline for **Fritz Haber (1868–1934)**,
built for a 1ère Générale Euro English class by Mesoud.

The project visualizes Haber's dual legacy — father of synthetic fertilizers and
father of chemical warfare — as a horizontally scrollable, drag-navigable timeline.

---

## File Structure

```
index.html        ← Entire project (HTML + CSS + JS in one file)
CLAUDE.md         ← This file
```

No build tools, no dependencies, no npm. Pure vanilla HTML/CSS/JS.

---

## Architecture

### Layout (CSS Flexbox)
```
body (flex column, height: 100vh, overflow: hidden)
├── .top-bar          — Fixed title/subtitle header
└── .timeline-wrapper — Horizontally scrollable viewport
    └── .timeline-content  — Absolutely positioned canvas (dynamic width)
        ├── <svg #connections> — SVG connector lines (z-index: 1)
        ├── .axis              — Dark horizontal bar at 50% height (z-index: 2)
        │   ├── .axis-dot      — Per-event tick mark (z-index: 3)
        │   └── .axis-label    — Year label
        └── .event-box(.top|.bottom)  — Event cards (z-index: 4)
```

### Positioning System
- **BASE_YEAR**: `1868.93` (birth year, used as X=0 anchor)
- **SCALE**: `60px per year` — fractional years map to sub-year positions
- **OFFSET_X**: `100px` left margin
- **Anchor formula**: `anchorX = OFFSET_X + (event.year - BASE_YEAR) * SCALE`
- **Box center**: `boxX = anchorX - 80` (boxes are 160px wide)

### Collision Resolution
Events on the same side (top/bottom) that would overlap are shifted right:
```js
if (curr.boxX < prev.boxX + 180) {
    curr.boxX = prev.boxX + 180;
}
```
This runs separately for `topEvents` (DESSUS) and `bottomEvents` (DESSOUS).

### SVG Connector Lines
Each event draws a straight `<line>` from the center of its box to its anchor dot on the axis. Lines are black, 1px, non-interactive (`pointer-events: none`).

---

## Data Format

Each event in `eventsData[]` uses this shape:

```js
{
  id: Number,
  year: Float,        // e.g. 1915.31 — used for precise X positioning
  side: 'DESSUS' | 'DESSOUS',  // DESSUS = above axis, DESSOUS = below
  date: String,       // Human-readable date shown in card header
  location: String,   // Shown in card header (italic)
  category: String,   // Determines card color (see categoryColors)
  text: String        // Card body content
}
```

### Category → Color Map
| Category     | Color   | Hex       |
|-------------|---------|-----------|
| Birth/Nobel  | Blue    | `#2196F3` |
| War/Tragedy  | Red     | `#F44336` |
| Career       | Green   | `#4CAF50` |
| Science      | Purple  | `#9C27B0` |
| Personal     | Orange  | `#FF9800` |
| Death/Exile  | Grey    | `#607D8B` |
| Marriage     | Pink    | `#E91E63` |
| Religion     | Brown   | `#795548` |

---

## Key Conventions

### Styling Rules
- **All text is bold** (`font-weight: bold` on `*`) — this is intentional by design
- No animations — hover only changes `border-width` from 2px → 4px (instant)
- No legend — categories are distinguished by color only
- White background (`#ffffff`) throughout
- Font: Arial, sans-serif

### No Legend
The project explicitly has **no legend**. Do not add one. Color meaning is
conveyed by context and card headers only.

### Rendering Flow
`renderTimeline()` is called on `load` and `resize`. It:
1. Clears all `.event-box`, `.axis-dot`, `.axis-label`, and SVG content
2. Recalculates `centerY` from current container height
3. Re-renders everything from `eventsData`

Do **not** cache DOM positions between renders — always recalculate from scratch.

### Scroll Interaction
- **Mouse drag**: `mousedown` → `mousemove` with 1.5× walk multiplier
- **Wheel**: `deltaY` mapped directly to `scrollLeft` (horizontal scroll)
- No touch events are implemented

---

## How to Add a New Event

1. Add an entry to `eventsData[]` with a unique `id` and a fractional `year`
2. Choose `side: 'DESSUS'` (above) or `'DESSOUS'` (below)
3. Use an existing `category` string, or add a new one to `categoryColors`
4. No other changes needed — layout recalculates automatically

---

## Common Tasks for AI Assistants

| Task | Where to look |
|------|--------------|
| Add/edit an event | `eventsData[]` array |
| Change a color | `categoryColors` object |
| Adjust spacing/scale | `SCALE`, `OFFSET_X`, `BASE_YEAR` constants |
| Change card width | `width: 160px` in `.event-box` CSS + `boxX = anchorX - 80` in JS |
| Fix overlapping boxes | Adjust the `180` threshold in `resolveCollisions()` |
| Change axis thickness | `.axis` height in CSS |
| Add touch scrolling | Extend the scroll event listeners section |

---

## Project Context

- **Subject**: Euro English, 1ère Générale
- **Author**: Mesoud
- **Topic**: Fritz Haber's dual legacy as both savior (Haber-Bosch process) and destroyer (chemical warfare)
- **Language**: English (academic/formal register)
- **No external dependencies** — must remain a self-contained single HTML file
