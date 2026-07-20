# C15.4 — The World Grid

> **The one-sentence version:** sections tile the map spatially, and the master file's per-section position
> data lets the streamer turn "the player is here" into "load these section ids" — a neighbourhood of the
> grid around the player.

[← C15.3 — Section residency & the stream file](03-residency.md) · [Chapter 15 hub](C15-Track-Streaming.md) ·
[Next: C15.5 — Visibility & culling data →](05-visibility.md)

---

## Sections are places

A streaming section is not an arbitrary bundle — it is a **region of the world**. The 720 sections
([C15.2](02-section-table.md)) tile the map so that any world position falls within some section, and nearby
positions fall in nearby sections. This spatial organisation is what makes streaming tractable: the set of
sections you need is simply the ones *around where you are*.

The master file carries the per-section **position/bounds** data (chunk `0x00034250` at 109 × 92, and the
section-data chunks `0x00034108`/`0x00034109` — [C15.1](01-master-layout.md)) that records where each section
sits. The recurring **92-byte** stride, shared with the section table, suggests these are parallel per-section
records keyed by the same index.

> ✅ *Verified:* the master file carries per-section position data (`0x00034250` = 109 × 92) alongside the
> 720-entry section table, both on a 92-byte stride.
> 🟡 *Reasoned:* that this data expresses a spatial grid the streamer queries by position is inferred from its
> per-section structure and the nature of open-world streaming; the chunks and their dimensions are verified.

## From position to sections

The streamer's core query is "given my world position, which sections are resident?" With a grid, this is a
neighbourhood lookup:

```
needed = { section s : distance(player, section_bounds[s]) < load_radius }
```

- **Position → cell.** The player's `(x, y)` maps to a grid cell (the world is Z-up —
  [C8.4](../C8-Geometry-Solids/04-bounding-boxes.md) — so streaming is driven by the horizontal position).
- **Cell → neighbourhood.** Take the cell and its neighbours out to the load radius.
- **Neighbourhood → section ids.** The position data maps cells/bounds to the section ids to load
  ([C15.3](03-residency.md)).

This is why the world can be huge yet cheap to query: you never consider all 720 sections, only the handful in
your vicinity.

## Why a grid

A regular spatial partition buys the streamer everything it needs:

- **O(1) needed-set computation.** From a position you compute the surrounding cells directly — no searching.
- **Predictable memory.** The number of resident sections is bounded by the neighbourhood size, so the memory
  budget is knowable and stable as you drive.
- **Smooth transitions.** Moving one cell over adds a strip of new sections and drops a strip behind — gradual
  churn, not a whole-world reload.

## Sections contain the world

Each section, once resident, holds the actual content of its patch of map — assembled from the formats earlier
in this book:

- **Geometry** — SolidLists of buildings, road, terrain ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)–[9](../C9-Meshes-FVF/C9-Meshes-FVF.md)).
- **Textures** — the section's TPKs ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).
- **Scenery and props** — instance placements and the cull tree ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)).

So the grid is the world's spatial index and the sections are its contents; driving through the city is
walking the grid, streaming each neighbourhood's sections as you go.

## Editing implications

- **Keep position data and the section table in sync.** They are parallel per-section records; edit a
  section's identity or existence and both must agree.
- **Respect spatial locality.** A section's content should match its place; putting distant geometry in a
  section breaks the "load nearby" assumption and causes pop-in or missing world.
- **Bounds must enclose content.** If you move geometry within a section, its recorded bounds must still
  contain it, or the streamer/culler mishandles it ([C15.5](05-visibility.md)).

---

### Key takeaways

- The 720 sections tile the map; nearby positions fall in nearby sections.
- Per-section position data (`0x00034250`, 92-byte stride) lets the streamer map position → needed section
  ids.
- A grid gives O(1) needed-set computation, bounded memory, and smooth strip-by-strip churn.
- Resident sections hold the patch's geometry, textures, and scenery — the earlier chapters' formats.
- Keep position data and the section table in sync, respect spatial locality, and keep bounds enclosing
  content.

**Continue:** [C15.5 — Visibility & culling data](05-visibility.md) · [Chapter 15 hub](C15-Track-Streaming.md)
