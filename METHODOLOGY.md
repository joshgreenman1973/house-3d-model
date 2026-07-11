# The house, in three dimensions — methodology

## What this is
An interactive 3D "dollhouse" model of a listed house (pending at $999,000; ~3,518 sq ft; 3 bedrooms, 5 baths per the listing), built with Three.js and viewable in any browser. Rooms connect through doorways, stairs run between floors and to-scale furniture can be placed in any room. Built July 10, 2026.

## Sources
- Three 2D floor-plan images from the property listing (lower level, main level, upper level), which include per-room dimension labels and a floor-selector sidebar stating total internal size (~3,518 sq ft) and room counts.
- One aerial photo of the house, used only as a sanity check on the overall footprint and the angle of the bedroom/garage wing.
- The owner's description that ceiling heights range from 8 to roughly 12–13 feet with lots of variation.

## Exact match to the listing plans (default view)
The default view — **"Match listing floorplans (exact)"** — is built directly from the three listing floorplan images (`plan-lower/main/upper.png`), not hand-traced:

1. **Walls from the drawing.** Each image's wall lines are the listing's own distinct navy color (RGB 31,68,148). A script (`plans_to_walls` steps, run once) detects those pixels into a wall mask (`wallmask-*.png`). The web app extrudes that mask into 3D — every wall pixel becomes a short vertical column — so the walls follow the drawing exactly, including door openings, the angled wing, and the curved rotunda. Nothing is placed by eye.
2. **The drawing itself is the floor.** The plan image is mapped onto each floor plate through the *same* transform used to place the walls, so walls and drawing register perfectly and every room label, dimension and fixture from the listing is visible on the floor.
3. **True scale.** Each floor's feet-per-pixel is calibrated from that drawing's own printed room dimensions (upper 0.0305, main 0.0673, lower 0.069 ft/px — the images are screenshots at different zooms), with aspect ratio preserved, so rooms and the placed furniture are to scale.

Unchecking the box switches to the interpreted hand-built massing model described below (varied ceiling heights, room-type floor colors, modeled fixtures/stairs) — useful for exploring, but the exact view is the one that matches the plans.

## How the traced model was built
- Each room is modeled as a rectangle using the dimensions printed on the floor plans (e.g. "Living Room 15'9" x 26'4""). Where no dimension was printed (halls, small closets, the primary bathroom's full extent), sizes were estimated visually from the plan images.
- Room positions were placed by eye to match the plan images, then checked by rendering the model in plan view and comparing it side-by-side against each floor-plan image.
- The angled wing (primary suite above; garage and workspace below) is modeled at a 32-degree rotation, estimated from the plan and aerial images.
- **Doorways**: door openings (with headers above) are cut into walls wherever the plans show rooms connecting. Exact door positions and widths are estimates read off the drawings; standard 6'10" door height assumed (7'6" for the garage vehicle door).
- **The rotunda is cylindrical**: the round tower joining the two wings is modeled as a true cylinder on both levels — curved walls with arched door openings, a full round floor on the lower level and a ring-shaped floor on the main level that the spiral stair rises through. Its ~11-foot diameter and exact position are estimated from the drawings, placed tangent to the primary bedroom's east wall (which it connects to directly). On its south side a short foyer connects the drum to the dining room (main level) and to the library (lower level); on the plan this junction is a curved glazed bay, simplified here to a straight vestibule.
- **The primary-suite deck** is modeled as the clipped pentagon shown on the plan, not a rectangle.
- **Bathroom fixtures and partitions**: the primary bath is divided into its tub nook and a walled water-closet (toilet room), and every bathroom is fitted with to-scale fixtures — tubs, showers with glass, toilets and vanities — plus a cedar bench in the sauna. Fixture positions are read off the plans and are approximate.
- **Dimensions on every room**: rooms show the size printed on the listing plan where one exists; rooms the plans left unlabeled (halls, the foyer, closets, stairs, landings) show a computed size prefixed with "~". The rotunda shows its approximate diameter. All labels also show the assumed ceiling height.
- **Fireplaces**: the four fireplaces marked on the plans (living room, game room, beside the lower rotunda at the library, and between the primary-suite deck and bedroom) are modeled as simple masonry masses at those locations.
- **Room names** follow the listing plans verbatim — including three rooms all labeled "Primary Bedroom" in the suite wing (the 30'0" × 27'8" main room plus the 12'11" × 13'0" and 9'6" × 10'2" rooms, shown here as "Bedroom" with their printed dimensions).
- **Stairs**: the plans show two straight runs in the main stack ("Stairs Down" and "Stairs Up" side by side near the kitchen) and a spiral stair in the round rotunda joining the two wings. Modeled as: a straight run from the lower level arriving at a small main-level landing; a second, switchback flight from that landing floating back over the first, arriving at the upper landing; and a spiral stair rising from the lower rotunda through a circular opening in the main-level slab. Tread counts and pitch are schematic — the flights are compressed to fit the stair rooms drawn on the plans.
- **Open to below**: the gray "Open To Below" area on the upper-floor plan is modeled as a true void — no floor, no walls — with a railing along the upper landing that overlooks it. Aligning the three plans by the stair column places this void over the kitchen area, so the kitchen is modeled double-height (~12 ft walls, open above). If the void actually sits over a different room, the alignment of the upper floor should be revisited.
- **Ceiling heights**: per room, ranging 8 to 13 ft. The listing plans give no heights; the assignment (living room ~13 ft, primary bedroom and kitchen ~12 ft, most other rooms 8–9.5 ft) is an educated guess from the "open to below," the fireplaces and the owner's description. Each room's label shows its assumed height.
- Levels are stacked at 10.5 ft floor-to-floor.

## Furniture
- The palette contains ~27 common pieces (beds, sofas, tables, pool table, car, washer, etc.) modeled at standard retail dimensions, which are shown next to each item (e.g. queen bed 5'0" × 6'8"; midsize car 6' × 15'2").
- Pieces are placed by clicking, moved by dragging, rotated in 45° increments and deleted; the layout persists in the browser's local storage (per device, not shared).
- Furniture footprints and the house share one scale, so fit checks ("does a king bed leave walking room in the small bedroom?") are meaningful to within the model's overall accuracy.

## Known limitations
- This is a massing study, not a measured drawing. Windows, closet fittings, stair railings, fireplaces and built-ins are omitted or simplified; door swings are not modeled.
- Most rooms are pure rectangles (the rotunda and the primary-suite deck are the exceptions). A few subtler shapes remain squared off: the angled bay corner of the primary bedroom, the rounded corner of the lower-level 7'8" × 6'2" storage room, and the curved glazed bay between the rotunda and the dining room/library (drawn here as a straight foyer). Adjacent rooms draw their own walls, so some shared walls render doubled.
- Stair pitch is steeper than code-compliant stairs because the flights are compressed into the stair footprints printed on the plans.
- Total modeled area will not sum exactly to the listed 3,518 sq ft (listings typically exclude garage, storage and decks; this model draws them all).

## Confidence
- Room names and printed dimensions: high (transcribed from the listing plans).
- Relative room positions and which rooms connect: medium (visual tracing, verified in plan view against the source images).
- Door positions, stair geometry, wing angle, ceiling heights, unlabeled room sizes: low (estimates).
- Furniture dimensions: high (standard retail sizes, labeled on each item).
