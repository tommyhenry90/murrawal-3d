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

## Decisions so far
- Living room: built-in bookcase on the north wall, opposite the upright piano on the south wall; grey sofa beside the bookcase; TV unit on the west wall.
- The flat sits under the west end of the house; the area under the deep part of the deck is an open covered terrace.
- The studio sits alongside the house's west side; the garage faces the street lane, north of Bed 3 (both from the aerial).
- Materials: cream face brick, white cladding on the yard side, brown concrete tiles with solar on the yard slope, beige metal skillion over the street side, dark clinker brick garage.

## Rules
- Keep it one self-contained HTML file.
- three.js stays pinned at 0.128.0 (UMD builds); no other external hosts.
- `<meta name="robots" content="noindex, nofollow">` keeps the address and floor plan out of search results.
