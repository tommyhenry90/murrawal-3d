# 8A Murrawal Road – 3D model

Single-file three.js model of the house at 8A Murrawal Road, Stanwell Park.
Open `index.html` in a browser to view it (needs internet: three.js r128 loads from jsDelivr).

Hosted on GitHub Pages — see the repo's Pages URL.

## Sources
Built from the Ray White floor plan, interior and exterior photos, and a drone aerial.
Room sizes follow the agent plan. Ceiling heights, roof pitch and exact outbuilding positions are estimates.

## Coordinates
- Metres. Origin = north-west corner of the house.
- x = east, z = south (towards the yard), y = up.
- Street-side ground `UP` = 2.2, main floor `FFL` = 2.7 (walls `GH` = 2.55), flat floor ≈ 0 (`LH` = 2.4).

## Code layout (search for the `// ----------` section markers)
- Procedural canvas textures (`TX`) → materials (`M`). `MG` / `ML` / `MO` are wall materials clipped for cutaway view (upper floor, flat, outbuildings).
- `box()` applies metre-based UVs so textures tile at real-world scale.
- `wall(parent, mats, axis, c, a, b, y0, h, t, openings, skin)`: axis `'x'` runs along x at z = c; `'z'` runs along z at x = c.
  Openings: `{ type: 'door' | 'extdoor' | 'window' | 'slider' | 'open' | 'garage' | 'cupboard', s, e, sill?, head? }`, with `s`/`e` measured along the wall axis.
  `skin` (`EXT(side, outsideMat, insideMat)`) marks an exterior wall and its finishes.
- Furniture helpers: `bed`, `sofa`, `chair`, `table`, `stool`, `toilet`, `car`.
- Roofs: `dutchRoof` (main house), `hipRoof`, `quad` (skillions), `solarArray`, `skylight`.
- Site: `bank`, `fence`, `bush`, `tree`, `palm`, `rock`.
- Groups: `site`, `ground` (upper floor + deck), `lower` (flat + covered terrace), `outb` (studio, garage), `roofMain`, `roofOut`.
- Room labels: `LABELS`. View modes: `apply()`. Camera framing: `viewFor()`.

## Play mode

The `Play` chip turns the model into a small Sims-style game. Click a sim to select them (green
plumbob), then click the floor to send them there; the panel on the right has their needs and
one button per interaction. `Free will` lets them look after themselves — when a need drops below
0.35 they walk to the matching object on their own.

- `SIMS` — the household: Tommy, Duncan, Sherin, Harriet and Emma upstairs, Mark and Shirley in
  the flat, and three labradors. Everyone belongs to one level and nobody uses the stairs.
- The dogs have temperaments rather than needs: `restless` (how often they want to be elsewhere),
  `speed`, `follow` (how much they stick to a person) and `laze` (how readily they settle), plus
  four social traits:
  - `playmate` — Marley and Cruise pair off and romp on the lawn (`chase()` picks points near each
    other out on the grass). A session runs 20-45 s, then `playCool` keeps them from restarting
    immediately. Maggie has no playmate, so she ignores the other two.
  - `fetch` — Maggie's instead: she picks a person, bolts 4-8 m away after the ball, brings it
    back, and repeats a few times.
  - `zoomies` — Cruise only. Brief bursts at five times his usual pace, likeliest mid-romp. He is
    the slowest dog on the property and by far the fastest for about a second at a time.
  - `bond` — Cruise's person is Shirley. He follows her across levels, and `sofaBeside()` watches
    for her sitting down and sends him to whichever nap spot is within 2 m of her seat. `eager`
    lets him trot at 2.4x while catching up with her — the one thing he hurries for.
  - Timers (`play`, `playCool`, `zoom`) tick at the top of `stepSim`, *before* the "already
    walking" early return. Put them in the idle branch and they only count down while a dog is
    standing still, which it almost never is: the romp then never ends.
- Everyone wanders the whole property, not just their own floor: `wander()` picks another level
  some of the time, and `Garden` is a `fun` spot on the lawn offered to upstairs people too via
  its `homes` field. Crossing the property takes 20 seconds or more, so the autonomy threshold is
  0.45 rather than 0.35 — set it lower and people start bottoming out needs while still walking.
  Someone queueing for a busy spot mills about on their current level instead of wandering off.
- `NAPS` — the sofas and beds the dogs sleep on, claimed one dog at a time like the human acts.
  A dog walks to the spot's `go` point and then hops up; `y` is the height of the surface, and the
  pose sits the dog 0.15 m above its own origin so its belly lands on the cushion rather than in it.
  With every spot taken they flop on the floor instead.
- An act is `taken` while someone is using it, and `pickAct` skips taken ones, so people queue for
  a bed or the bathroom instead of standing inside each other. A player-issued order overrides it.
- The garage has a small gym — squat rack, bench, dumbbells, wall mirror — and `Weights` is an
  act with `only: 'Tommy'`, so it is offered to nobody else. `pickAct` picks a person's own `only`
  spot 70% of the time, which is what makes him go regularly. The `lift` pose swings both arms
  through about 110 degrees and shows a barbell parented to one arm pivot, so it tracks both hands.
- `brawn` on a figure broadens the frame — chest, shoulders, arms and legs scale, height does not.
  Default 1; Tommy is 1.45. It is purely cosmetic: everyone navigates on the same `SIM_R`.
- Faces and clothes come from the photos in `characters/`, which is git-ignored — the likenesses
  are blocky approximations in code and the photos themselves are never published.
- `ACTS` — the interactions. Each has `go` (where the sim stands to begin) and `pose`
  (`stand` / `sit` / `lie`, with the exact spot and facing it ends up in). Sims with the same need
  are spread across matching acts by `slot`, so they don't all pile onto one bed.
- `NEEDS` — energy, hunger, bladder, fun, each draining at its own rate.

### Navigation

There is no hand-drawn walkable map. `floor()` records every floor rect and `box()` / `cyl()`
record every solid into `NAV`, so the grid is derived from the geometry and stays correct when
you move a wall. `buildNav()` turns that into a 10 cm occupancy grid per level; sims path with A*
over it and string-pull the result so they cut corners naturally.

There are three levels: `g` (upper floor and deck, at `FFL`), `l` (the flat *and the rear yard* —
the yard is 15 mm below the flat, so they share one grid and the flat's slider is just an opening
in it) and `s` (the street side at `UP`: courtyard, parking, front garden, side passages and the
lane). Solids are global and selected by height, so the same list serves every level — the house
slab blocks the street grid, the garden hedge blocks the yard grid, and so on.

`PORTALS` are the stairs and steps, each an explicit `[x, y, z]` polyline between two level grids:
the deck stairs, the front steps, Bed 3's side door and the side-passage steps. `planTo()` runs a
breadth-first search over that portal graph, then builds a plan alternating grid walks and portal
walks. A character follows portal points with their real heights, so they walk *down* the stairs
rather than dropping off the end. Characters live in the `folk` group rather than under a floor,
because they change level as they go; `poseSim` sets their visibility from the floor filter each
frame, and outdoor levels are always shown.

Things worth knowing before you change it:

- A cell is blocked if a solid overlaps the **walking band**, `baseY + 0.2` to `baseY + 1.0`.
  The lower bound clears floor steps (the flat's bathroom is 165 mm up); the upper bound sits
  under door heads so doorways stay open.
- Solids are inflated by the sim's radius, so a walkable cell is a legal position for a sim's
  centre. Walls use the full `SIM_R` (0.22); furniture uses `SOFT_R` (0.12), because at a full
  body width the TV unit and piano seal the living room door and the dining chairs cut the dining
  room in half. Sims will clip a chair by a few centimetres — that is the trade.
- Transparent materials never block, so glass and sliders are walk-through. Door leaves are drawn
  shut but wrapped in `NAV.skip` so they don't block either.
- The deck, the flat's stepped bathroom and all the outdoor ground aren't laid with `floor()`, so
  they are pushed into `NAV.floors` by hand. Rects must *overlap* at thresholds: the deck rects
  start at z 11.55 and the rear lawn at z 11.5, because a floor ending at z 11.625 and another
  starting at z 11.725 leaves a 10 cm hole and the two never connect.
- The boundary hedge used to start at x 12.65, hard against the deck stairs. The four lowest
  treads block the side path at walking height, leaving a 23 cm gap — so the foot of both
  staircases was sealed off from the lawn. The hedge now starts at x 13.2 and the path is clear.

### Controls on small screens

The play panel is a bottom sheet on phones and a top-right card everywhere else, and it collapses
to just the name tabs and a status line. It starts collapsed the first time you hit Play on a
small screen. Measured share of the viewport taken by all the UI:

| | phone portrait | phone landscape | desktop |
|---|---|---|---|
| play off | 13% | 17% | 8% |
| collapsed | 21% | 20% | 11% |
| expanded | 26% | 27% | 14% |

Two things to keep in mind if you touch this:

- `resize()` publishes the real control-bar height as `--ctrl-h`, and the phone sheet sits at
  `bottom: calc(var(--ctrl-h) + 16px)`. The bar wraps to different heights, so don't hardcode it.
- The short-screen rule must set `bottom: auto` on the panel. It keeps `top` from the base rule,
  and setting only `bottom` gives it both — the panel then stretches the full height and swallows
  94% of a landscape phone.

## Decisions so far
- Living room: built-in bookcase on the north wall, opposite the upright piano on the south wall; grey sofa beside the bookcase; TV unit on the west wall.
- The flat sits under the west end of the house. The rest of the under-house space is sealed sub-floor —
  brick on all three open faces up to the slab soffit, no door or vent. There is no access under there.
  The paving under the deck (z > 11.725) is still open and walkable.
- No palms in the yard. The `palm()` helper is still defined but now unused.
- The studio sits alongside the house's west side; the garage faces the street lane, north of Bed 3 (both from the aerial).
- Materials: cream face brick, white cladding on the yard side, brown concrete tiles with solar on the yard slope, beige metal skillion over the street side, dark clinker brick garage.

## Rules
- Keep it one self-contained HTML file.
- three.js stays pinned at 0.128.0 (UMD builds); no other external hosts.
- `<meta name="robots" content="noindex, nofollow">` keeps the address and floor plan out of search results.
