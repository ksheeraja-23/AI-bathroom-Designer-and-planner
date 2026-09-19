# KOHLER Studio — AI Bathroom Designer & Planner

A working prototype for **Track 1: KOHLER AI Bathroom Designer & Planner**.

## What it does

The customer enters a room size (ft × ft), a budget, and an aesthetic theme
(Minimalist Modern / Classic Luxury / Japanese Zen). The app then:

1. **Optimizes a fixture bundle** — a multiple-choice knapsack picks the
   highest style/quality/water-efficiency score per category (toilet,
   faucet, vanity, shower system, optionally a tub and mirror) that fits
   the budget, after first filtering out fixtures whose footprint won't
   reasonably fit the room.
2. **Checks the fit** — total fixture floor area is compared against a
   usable-area estimate for the room (reserving space for clearances and
   circulation), with a clear "fits comfortably" / "tight fit" badge.
3. **Draws a schematic 2D floor plan** (SVG) placing the chosen fixtures
   against the walls, scaled to the actual room dimensions.
4. **Quantifies water savings** versus pre-efficiency baseline fixtures
   (gallons/year and % reduction) — ties the recommendation back to
   KOHLER's sustainability commitments.
5. **Generates a written design rationale**, either live via the Anthropic
   API (if you supply a key) or a rule-based fallback so the app is fully
   functional with zero external dependencies.

## Run it

No build step, no server required.

```bash
open index.html        # macOS
# or just double-click index.html / drag it into a browser tab
```

It's a single static HTML file — open it directly in any modern browser.

### Optional: live AI rationale

Click "Add an Anthropic API key" in the left panel and paste a key
(`sk-ant-...`). The app calls `api.anthropic.com/v1/messages` directly from
the browser using the `anthropic-dangerous-direct-browser-access` header.
This is fine for a demo; for a real deployment, proxy that call through a
small backend so the key never touches the browser. Without a key, the app
falls back to a rule-based rationale generated from the same bundle data.

## Architecture

```
index.html
├── CATALOG            static product data: price, style tags, footprint,
│                       water usage (gpf/gpm), premium flag
├── buildBundle()       constraint filtering (spatial) + multiple-choice
│                       knapsack (budget) -> chosen bundle
├── sustainabilityStats() water-usage math vs. baseline fixtures
├── renderFloorplan()   heuristic wall-based SVG layout scaled to room ft
└── generateRationale() Anthropic API call with rule-based fallback
```

Everything runs client-side in vanilla JS — no framework, no build tools —
so it's trivial to open, read, and extend.

## Known limitations / next steps

- Floor plan placement is a heuristic (fixtures snapped to walls/corners),
  not a true bin-packing or CAD-accurate layout. A real version would take
  a door/window position and run proper rectangle packing, or accept a
  room photo and use vision to infer layout constraints.
- Catalog is a small illustrative dataset (24 SKUs across 6 categories,
  modeled on real KOHLER product lines) rather than a live product feed.
- No true 3D rendering — the 2D top-down plan was prioritized as the more
  reliable and higher-signal deliverable for this timeline; a Three.js
  3D view is a natural follow-up.
- Water-savings math uses fixed occupancy/usage assumptions (2 occupants,
  typical flush/faucet/shower frequency) rather than user-entered habits.
