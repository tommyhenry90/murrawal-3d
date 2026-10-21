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

- `SIMS` — the five characters, four upstairs and one in the flat. A sim belongs to one level and
  never uses the stairs.
- `ACTS` — the interactions. Each has `go` (where the sim stands to begin) and `pose`
  (`stand` / `sit` / `lie`, with the exact spot and facing it ends up in). Sims with the same need
  are spread across matching acts by `slot`, so they don't all pile onto one bed.
- `NEEDS` — energy, hunger, bladder, fun, each draining at its own rate.

### Navigation

There is no hand-drawn walkable map. `floor()` records every floor rect and `box()` / `cyl()`
record every solid into `NAV`, so the grid is derived from the geometry and stays correct when
you move a wall. `buildNav()` turns that into a 10 cm occupancy grid per level; sims path with A*
over it and string-pull the result so they cut corners naturally.

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
- The deck and the flat's stepped bathroom aren't laid with `floor()`, so they are pushed into
  `NAV.floors` by hand. The deck rects start at z 11.55 to overlap the slider threshold — without
  that overlap there is a 10 cm gap at z 11.625 and the deck is unreachable.

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
