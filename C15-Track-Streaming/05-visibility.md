# C15.5 — Visibility & Culling Data

> **The one-sentence version:** residency decides what's *in memory*; visibility decides what's *drawn* — the
> per-section **scenery cull tree** (Chapter 16) and the section-level potentially-visible set record what can
> be seen from where, so the renderer culls whole sections and objects that can't appear on screen.

[← C15.4 — The world grid](04-world-grid.md) · [Chapter 15 hub](C15-Track-Streaming.md) ·
[Next: C15.6 — Working with track data safely →](06-editing-track.md)

---

## Two different questions

Streaming and visibility answer distinct questions, and conflating them is a common confusion:

- **Residency** ([C15.3](03-residency.md)): *is this section in memory?* Driven by distance on the grid
  ([C15.4](04-world-grid.md)).
- **Visibility**: *should this resident section (or object) be drawn this frame?* Driven by what the camera
  can actually see.

A section can be resident but not visible (behind you, or occluded by a building), and the renderer must skip
it to save time. The master file's visibility data is what makes that decision fast.

## Where visibility actually lives

A correction to a tempting misreading: the large `0x0003B800`/`0x8003B810`/`0x8003B900` blocks are **not** the
visibility data — `0x0003B800` is the CARP road network and the `0x8003B8xx`/`0x8003B900` blocks are
event-sequence data ([C15.1](01-master-layout.md), [Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)).
Visibility is handled at two levels that this book verifies elsewhere:

- **Per-section spatial culling** — the **scenery cull tree** ([Chapter 16](../C16-Scenery-Cull/05-cull-tree.md)),
  a per-section 36-byte-node AABB hierarchy that decides which props are potentially visible from where. In the
  master file, the recursive `0x8003B6xx` node family is the spatial-tree data of this kind.
- **Coarse section culling** — the streamer/renderer skips resident sections that can't be seen, using section
  bounds ([C15.4](04-world-grid.md)).

Precomputing this offline (the `PrecullerBooBooScript.hoo` file beside the track is the preculling step) trades
build time for runtime speed — essential for a dense city where most of the world is hidden behind nearer
buildings.

> ✅ *Verified:* a recursive `0x8003B6xx` node hierarchy exists in the master file; the per-section scenery
> cull tree (36-byte nodes) is decoded in [Chapter 16](../C16-Scenery-Cull/05-cull-tree.md); a preculler script
> accompanies the track. `0x0003B800` is CARP (Chapter 18), not visibility.
> 🟡 *Reasoned:* the precise division between the master file's `0x8003B6xx` tree and per-section trees is
> identified by structure; the cull-tree format itself is verified.

## Why precompute visibility

Real-time occlusion in a dense city is expensive; precomputing it is the classic answer:

- **Most of the world is hidden.** From any street, buildings occlude the vast majority of the city; drawing
  only the visible slice is a huge saving.
- **Sections bound the problem.** Computing visibility *between sections* (coarse) plus per-object culling
  within visible sections (fine) is far cheaper than per-object visibility across the whole map.
- **Offline is free at runtime.** The preculler computes the PVS once at build time; the game just looks it
  up.

This is why streaming and visibility are separate systems working together: streaming keeps a neighbourhood
resident; visibility ensures only the seen fraction of that neighbourhood is drawn.

## The draw pipeline

Putting residency and visibility together, a frame draws the world like this:

```
resident sections (C15.3)
  → for the camera's section, look up its potentially-visible set (this chapter)
  → for each potentially-visible section, cull its objects against the frustum + cull tree (C16)
  → draw what survives
```

The section-level PVS is the coarse filter; the per-object **cull tree**
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) is the fine filter. Two stages, coarse to fine, so the
renderer spends its time only on things that might actually be on screen.

## Editing implications

- **Move geometry, and visibility may go stale.** The PVS was computed for the original layout; large changes
  (new occluders, moved buildings) can make it wrong, causing objects to vanish (culled when they shouldn't
  be) or over-draw.
- **Rebuild visibility for major edits.** Small tweaks are usually fine; structural changes ideally re-run the
  preculling step so the PVS matches.
- **Bounds must be honest.** Culling relies on correct section and object bounds
  ([C15.4](04-world-grid.md), [C16](../C16-Scenery-Cull/C16-Scenery-Cull.md)); stale bounds break it.

---

### Key takeaways

- Residency (in memory) and visibility (drawn) are separate systems; a section can be resident but not
  visible.
- Visibility lives in the per-section **scenery cull tree** (Chapter 16) and the section-level PVS — *not* the
  `0x0003B800` block, which is the CARP road network (Chapter 18).
- Visibility is precomputed offline (the preculler) to make dense-city rendering cheap at runtime.
- The pipeline is coarse-to-fine: section-level PVS, then the per-object cull tree (Chapter 16), then draw.
- Major geometry edits can stale the PVS; rebuild visibility and keep bounds honest.

**Continue:** [C15.6 — Working with track data safely](06-editing-track.md) · [Chapter 15 hub](C15-Track-Streaming.md)
