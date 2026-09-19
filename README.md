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
4. **Renders an interactive 3D room** (Three.js, drag-to-rotate /
   scroll-to-zoom) built from the exact same fixture placement as the 2D
   plan, with actual fixture geometry rather than placeholder boxes: a
   lathe-turned toilet bowl with tank and seat ring, a vanity with a
   vessel basin and gooseneck faucet, a glass-enclosed shower with a
   showerhead/arm/valve, an extruded soaking tub with a filled-looking
   interior, a brass-framed mirror with a warm sconce light, a towel bar,
   and a tiled floor — all procedurally generated, no external model or
   image files.
5. **Quantifies water savings** versus pre-efficiency baseline fixtures
   (gallons/year and % reduction) — ties the recommendation back to
   KOHLER's sustainability commitments.
6. **Generates a written design rationale** — attempts a live call to the
   Anthropic API, and falls back to a rule-based write-up built from the
   same bundle data if that call isn't available, so the app is always
   fully functional.

## Run it

No build step, no server required.

```bash
open index.html        # macOS
# or just double-click index.html / drag it into a browser tab
```

It's a single static HTML file — open it directly in any modern browser.

### About the AI rationale call

Clicking "Explain this bundle" calls `api.anthropic.com/v1/messages`
directly from the browser with no key attached. That succeeds when the
page is rendered inside a Claude artifact sandbox (which proxies the
call), and fails harmlessly everywhere else — the app then falls back to a
rule-based rationale built from the same bundle data, so the feature never
breaks the page. For a production deployment, route this call through a
small backend that holds the API key server-side.

## Architecture

```
index.html
├── CATALOG            static product data: price, style tags, footprint,
│                       water usage (gpf/gpm), premium flag
├── buildBundle()       constraint filtering (spatial) + multiple-choice
│                       knapsack (budget) -> chosen bundle
├── computeLayout()     shared feet-space fixture placement, used by both
│                       the 2D plan and the 3D scene so they never diverge
├── renderFloorplan()   computeLayout() -> scaled SVG top-down plan
├── buildToilet() / buildVanity() / buildTub() / buildShower() /
│   buildMirror() / buildTowelBar()
│                       procedural Three.js geometry for each fixture type
│                       (LatheGeometry bowls, ExtrudeGeometry tub shell,
│                       vessel basin, glass shower enclosure, etc.),
│                       authored at a base size and scaled to the chosen
│                       product's real footprint
├── render3D()          computeLayout() -> positions/scales each fixture
│                       group into a Three.js scene with OrbitControls
├── sustainabilityStats() water-usage math vs. baseline fixtures
└── generateRationale() Anthropic API call with rule-based fallback
```

Everything runs client-side in vanilla JS — no framework, no build tools —
so it's trivial to open, read, and extend. Three.js and its OrbitControls
addon load from a CDN (`cdn.jsdelivr.net`); the app degrades gracefully to
a text notice in the 3D panel if that CDN is unreachable, while the 2D
plan keeps working regardless.

## Known limitations / next steps

- Fixture placement is a heuristic (vanity/toilet/tub/shower snapped to
  walls and corners), not a true bin-packing or CAD-accurate layout. A
  real version would take a door/window position and run proper rectangle
  packing, or accept a room photo and use vision to infer constraints.
- 3D fixtures are procedurally built approximations of real Kohler
  silhouettes (lathe-turned toilet bowl, vessel basin, extruded tub
  shell), scaled to match the chosen product's real footprint — not
  imported CAD models, so fine details (branding, exact curvature) aren't
  reproduced.
- Catalog is a small illustrative dataset (24 SKUs across 6 categories,
  modeled on real KOHLER product lines) rather than a live product feed.
- Water-savings math uses fixed occupancy/usage assumptions (2 occupants,
  typical flush/faucet/shower frequency) rather than user-entered habits.
