# house-3d-model — handoff

Interactive 3D model of a specific house's interior, reconstructed from its
real-estate listing floorplans. Built to be viewed in any browser.

- **Live:** https://joshgreenman1973.github.io/house-3d-model/
- **Repo:** `joshgreenman1973/house-3d-model` (public, GitHub Pages from `main`, root)
- **Gallery:** listed in the Experiments gallery **personal** section
  (via `project-overrides.json` → `"audience": "personal"` in the parent repo).
- **Local dev:** any static server, e.g. `python3 -m http.server 8214` in this folder.
  There is a `house-3d` entry in `../.claude/launch.json`, but that server tends to
  die between sessions — a plain background `http.server` is more reliable.

## The house
10 Abbott Lane, Cornwall-on-Hudson, NY. Listing: pending $999,000, ~3,518 sq ft,
3 bed / 5 bath. Three levels:
- **Lower** (`plan-lower.png`): garage, workspace, library, game room, laundry, covered deck.
- **Main** (`plan-main.png`): angled primary-suite wing (bath, sauna, bedrooms, deck)
  joined by a round rotunda to the main block (dining, screened porch, kitchen,
  sitting room, living room, decks).
- **Upper** (`plan-upper.png`): two bedrooms, two baths, closets, bonus room, and the
  "Open To Below" void over the kitchen.

## plan.html — the flat furniture planner (Sep 2026)
`plan.html` is a second, self-contained page: the three plan images drawn top-down on
a 2D canvas with a furniture palette. No Three.js, no build. It exists because placing
furniture in the 3D view was fiddly; this is the utilitarian tool.
- Same `PLANS` calibration and plan-feet coordinates as `index.html`, so both pages
  read and write one layout. Storage key `abbott-plan-v1` is the planner's own
  (pieces + custom sizes); on every save it mirrors catalog pieces into
  `house3d-furniture` (the 3D model's key, `rotation.y = -screenAngle`).
- Walls come from the same `wallmask-*.png` files. They drive the gap readouts
  (rays from each side of the selected piece to the nearest wall or piece) and the
  amber "in a wall" outline.
- Room labels the listing's photo markers covered are re-lettered from the `RELABEL`
  table (image-pixel positions). The marker discs themselves were painted out of the
  plan PNGs (Sep 2026; originals were the same images with blue photo dots).
- Extras: snap (1"/3"/6"/1'), 1-ft grid, measure tool (length + angle; `A` rotates the
  selection to the measured line), custom pieces kept in the palette, undo/redo,
  JSON save/load, layout-in-URL link, PNG export at plan resolution.
- **Labels and rooms.** Every piece of text in the three plan PNGs was painted out
  (Sep 2026; the seller's photo markers sat on half the labels, and the leftovers looked
  bad). `LABELS` in `plan.html` re-letters all of it on the canvas at the original
  image-pixel positions: room names with the listing's printed dims, plus small tags
  (Fireplace, Closet, Stairs, Entry, Open to below). Labels hide when a piece sits on
  them and drop out at low zoom. `ROOMS` is derived from `LABELS` (anything with a size);
  `roomShape()` finds a room's outline by casting rays from its label to the walls in the
  wall mask along the room's axes (five parallel rays each way, every wall they meet),
  then picks the wall pair whose span matches the printed size and clamps open sides
  (kitchen, living room) to it. Measured outlines land within inches of the printed dims
  on all three floors. The wing angle is 24°, measured from the wall pixels (the 3D
  model's hand estimate was 32°). The toolbar's room menu frames a room and veils the
  rest; new pieces drop in its middle.
- **Listing import.** Paste a product URL: the page is fetched through `r.jina.ai` (a
  public reader; no key, no cost) and `parseDims()` reads width/depth/height out of the
  text — labeled fields, `84"W x 38"D x 34"H` forms, `W x D x H` order hints, cm/mm,
  fractions, feet-and-inches. Most big furniture stores (Wayfair, IKEA, Article, Pottery
  Barn product pages, CB2) block outside readers, so the reliable path is the paste
  box: select-all/copy on the product page, paste, same parser. Imported pieces keep
  `h` and `url`; the inspector links back to the listing. `allorigins.win` was tried as a
  second reader and dropped: no CORS header.
- Plan PNG cleanup (Sep 2026): all text and the photo-marker discs painted out (each
  pixel copied from the nearest clean floor pixel on its row; walls and deck boards
  left alone), lower-level aerial background made transparent. Originals are in git
  history (commit 210a0dc) if the erasure ever needs redoing.

## No build step
Everything is one file: **`index.html`** (HTML + CSS + a single ES-module `<script>`).
Three.js r160 is loaded from unpkg via an import map. Editing = edit `index.html`,
reload. There is nothing to compile or bundle.

## Two viewing modes
The **"Match listing floorplans (exact)"** checkbox (on by default) switches between:

1. **Exact mode (default, `overlayOn = true`)** — the deliverable. The actual listing
   plan images are laid on each floor plate, and the walls are *extruded from the plan
   drawings themselves*, so the model matches the floorplan by construction. All the
   hand-traced geometry is hidden.
2. **Interpreted massing model (`overlayOn = false`)** — the earlier hand-built model
   (rooms defined in the `LEVELS` array), with per-room ceiling heights, room-type floor
   colors, modeled fixtures, and simple stairs. Kept as an alternate view.

`setOverlay(on)` is the switch. It shows/hides both sets of geometry (see below).

## How "exact mode" works (the important part)
Each floor is an image, mapped into the shared plan coordinate system by an entry in
the **`PLANS`** object:

```js
PLANS = { 0:{file, imgW, imgH, w, h, cx, cy, level:0}, 1:{...}, 2:{...} }
```

`w/h` are the plane's size in **feet**, calibrated from each drawing's own printed room
dimensions (upper ≈ 0.0305, main ≈ 0.0673, lower ≈ 0.069 ft/px — the images are
screenshots at different zooms), aspect preserved. `cx/cy` position the plate.

The single most important helper:
```js
pxToPlan(p, px, py) -> [planX, planZ]     // image pixel -> plan (world XZ) coords
```
Both the floor **texture** and the extruded **walls** use this same mapping, so walls
sit exactly on the drawing. Everything downstream (windows, stairs, rotunda glass) is
positioned by mapping known image pixels through `pxToPlan`.

### Derived assets (generated by one-off Python)
The Python that made these used **PIL + numpy + scipy**. Regenerate whenever the plan
images change. Key trick: the listing draws all walls in one navy color **RGB (31,68,148)**.

- **`wallmask-<floor>.png`** — downsampled binary mask of navy wall pixels. The app
  extrudes one short box per set pixel (`InstancedMesh`) at `WALLH_IMG = 9 ft`. Cells
  within `ROT_R` of the rotunda center render as **glass** instead of solid (see below).
  Deck outer edges and stray isolated components are removed from this mask.
- **`railmask-<floor>.png`** — the deck **outer (woods-facing) edges** that were removed
  from the wall mask; the app extrudes these as low (`RAIL_H = 3 ft`) wood railings so
  decks read as open, not walled.
- **`windows.json`** — `{ "0":[[x0,y0,x1,y1],…], "1":[…], "2":[…] }`, window openings in
  MAIN/UPPER/LOWER **image pixels**. The app builds a glass pane + sill + header per rect.
- **Floor textures (`plan-*.png`, RGBA)** were edited in place:
  - aerial-forest background made transparent (kept only on the **lower** level so the
    house looks grounded; main/upper are clean cutouts);
  - hatched **decks/porches repainted as wood planks**;
  - the **"Open To Below"** void cut transparent (upper) so you see down to the kitchen;
  - **stairwell holes** cut in the **main** plate at the rotunda and the "Stairs Down"
    box so the spiral and down-flight show through instead of hiding under the plate.

### Cache-busting
`const AV = '?v=NN'` near the top is appended to every dynamically-loaded asset
(`plan-*.png`, `wallmask-*`, `railmask-*`, `windows.json`). **Bump `AV` whenever you
regenerate any of those images/JSON**, or browsers will serve stale copies. (The page
HTML itself is fetched fresh on deploy; only these sub-assets need the version query.)

## Feature map (all in `index.html`)
- **Walls** — `buildPlanFloor()` loads `wallmask`, extrudes solid walls + rotunda glass.
- **Rotunda glass** — cells within `ROT_R = 7.2 ft` of `ROT_C = [-0.41, -6.6]` (plan ft)
  render with `rotGlassMat` (translucent) instead of `imgWallMat`. This is why the tower
  is see-through rather than a closed drum.
- **Deck railings** — `railmask` → low `railWoodMat` boxes.
- **Windows** — `windows.json` → `glassMat` panes with opaque sill/header (`SILL_H`,`HEAD_H`).
- **Stairs** — `straightFlight(box, level, ascendZ)` builds two straight open-tread runs
  (Down in level 0, Up in level 1); `spiralStair(centerPx, level, rad)` builds the rotunda
  spiral in level 0. All footprints are given in MAIN-plan image px and mapped via
  `pxToPlan(PLANS['1'], …)`.
- **Furniture** — palette on the right; click a piece then click a floor to place it;
  drag to move, R to rotate, Delete to remove. Persists in `localStorage`
  (`house3d-furniture`). Raycasts onto the plan planes in overlay mode, room slabs otherwise.
- **Floor toggles / exploded view / plan-elevation-perspective views** — bottom-left panel.
- **AI-caution** popover and `METHODOLOGY.md` document sources, assumptions, and confidence.

### setOverlay visibility (gotcha)
Anything that must show in exact mode is registered so `setOverlay` can toggle it:
`planMeshes` (textures), `planWalls`, `planRotGlass`, `planRails`, `planWindows`,
`planStairs`, plus it hides the hand-traced materials (`handMats()`, `edgeMat`, slabs).
**Objects live inside `levelGroups[level]`; if a level group is hidden (single-floor
view), its children don't render regardless of their own `.visible`.** That's why
cross-floor stairs are placed in a specific level and the spiral/down-flight rely on the
cut holes to be seen in the all-floors default view.

## Deploying an update
```
# edit index.html and/or regenerate assets (bump AV if assets changed)
git add -A && git commit -m "…" && git push        # Pages redeploys from main
# verify (Pages takes ~1 min; add a cache-buster to the URL):
curl -s "https://joshgreenman1973.github.io/house-3d-model/index.html?cb=$RANDOM" | grep …
```
Deploys are under the **joshgreenman1973** account (`gh auth`). This is a personal
project — the repo is public (Pages requires it), but it's gated in the gallery listing.

## Known approximations / open items
- **Inter-floor registration** is by shared features (the upper floor was placed by
  aligning its "Open To Below" void over the kitchen; main/lower share a footprint).
  It's close but not survey-exact — a floor can be a couple of feet off.
- **Straight-stair slope** is steep: the plan draws the flights in a short, compressed
  footprint, so a single straight run climbing a full 10.5 ft floor is >45°. Kept straight
  to match the drawing. Real construction there likely has a longer run or a mid-landing
  the plan doesn't show. Options if revisiting: extend the run (overlaps rooms), add a
  landing, or reduce `LEVEL_GAP`.
- A few curved shapes are squared off; some shared walls render doubled; fixture and
  window placements are approximate. See `METHODOLOGY.md` for the full confidence rundown.

## Files
- `index.html` — the whole app.
- `plan-lower.png` / `plan-main.png` / `plan-upper.png` — floor textures (edited RGBA).
- `wallmask-*.png`, `railmask-*.png`, `windows.json` — derived geometry inputs.
- `plan-overview.png` — the listing's floor-selector thumbnail (not used by the app).
- `METHODOLOGY.md` — sources, assumptions, confidence levels.
- `HANDOFF.md` — this file.
