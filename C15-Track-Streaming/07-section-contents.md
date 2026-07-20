# C15.7 — The Anatomy of a Stream Section

> **The one-sentence version:** a stream section is not one thing but a *bundle* of co-located chunks for one tile of
> the world — render geometry (`SolidList`), textures (`TPK`), scenery (`0x34101/2/3`), and collision (terrain
> `0x34159`, walls `0x3B801`, smackables `0x34027`, FX emitters `0x3BC00`) — all streamed in together and all obeying
> the 16-byte record-alignment invariant.

[← C15.6 — Working with track data safely](06-editing-track.md) · [Chapter 15 hub](C15-Track-Streaming.md) ·
[Next: Chapter 16 — Scenery, Props & the Cull Tree →](../C16-Scenery-Cull/C16-Scenery-Cull.md)

---

## A section is a bundle

The section table ([C15.2](02-section-table.md)) indexes 720 sections; residency ([C15.3](03-residency.md)) streams
their blobs from `STREAML2RA.BUN`. But *what is in a blob?* A stream section is not a single asset — it's a **bundle
of co-located chunks**, everything the engine needs to present one tile of Rockport: the geometry you see, the
textures on it, the props scattered across it, and the collision you drive on and crash into. Streaming a section
pulls *all* of it in at once, so the tile arrives complete — visuals and physics together.

Decoding the section blob against the retail world, the top-level chunks group into four roles:

| Role | Chunk(s) | Decoded in |
|---|---|---|
| **Render geometry** | `SolidList` (solid objects, meshes) | [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md) |
| **Textures** | `TPK` texture pack | [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md) |
| **Scenery** | `SceneryHeader` `0x34101`, `SceneryInfos` `0x34102`, `SceneryInstances` `0x34103` | [Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md) |
| **Terrain collision** | `0x00034159` (ground-height mesh) | [C63.7](../C63-Collision-World/07-terrain-collision.md) |
| **Wall/object collision** | `0x0003B801` (`WCollisionPack`) | [C63.8](../C63-Collision-World/08-wcollisionpack.md) |
| **Smackable props** | `0x00034027` (knock-down spawners) | [C63.9](../C63-Collision-World/09-smackables-emitters.md) |
| **FX emitters** | `0x0003BC00` (emitter placements) | [C63.9](../C63-Collision-World/09-smackables-emitters.md) |

Most sections carry roughly one of each collision chunk (one terrain mesh, one `WCollisionPack`, one smackable pack),
alongside their geometry, textures, and scenery. The section is the *unit of world residency*
([C38.2](../C38-Resource-Streaming-Residency/02-sections-residency.md)): these chunks live and die together as the
player drives in and out of the tile.

> ✅ *Verified:* section blobs decode into `SolidList` + `TPK` + `ScenerySection` (`0x34101/2/3`) + terrain collision
> (`0x34159`) + `WCollisionPack` (`0x3B801`) + smackables (`0x34027`) + emitter placements (`0x3BC00`), each chunk's
> ID confirmed by its `speed.exe` loader dispatch (`cmp dword [reg], <id>`), routed through the dispatcher at
> `0x45D600` ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)).

## The visual and the physical, co-located

The section's grouping reveals a design principle: **the visual world and the physical world are separate data,
streamed together.** For one tile you get:

- **What you see** — the `SolidList` meshes ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) and their
  `TPK` textures ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), plus the scenery instances
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) placed across it.
- **What you touch** — the terrain height-field the wheels ride ([C63.7](../C63-Collision-World/07-terrain-collision.md)),
  the walls that stop the car ([C63.8](../C63-Collision-World/08-wcollisionpack.md)), and the props you knock down
  ([C63.9](../C63-Collision-World/09-smackables-emitters.md)).

They are *deliberately distinct* — the collision terrain is a coarse soup, not the render mesh
([C63.7](../C63-Collision-World/07-terrain-collision.md)); the `WCollisionPack` walls are corridor edges, not the
building meshes ([C63.8](../C63-Collision-World/08-wcollisionpack.md)). But they're *co-located in the same section
blob*, so a tile's visuals and physics always arrive as a matched set. This is why the world never streams in a
building you can see but drive through, or collision with no mesh: the section bundles both.

## The alignment invariant binds the bundle

Every record-bearing chunk in the bundle obeys the same rule the collision loaders enforce
([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)): after the 8-byte chunk header, `0x11` padding pushes
the first record onto a **16-byte boundary** — `(ptr + 0x17) & ~0xF`. Scenery instances
([C16.3](../C16-Scenery-Cull/03-instance-record.md)), terrain triangles, smackable records, and collision articles
all start 16-aligned; the whole section is **2048-aligned** in the stream ([C15.3](03-residency.md)).

This is what makes section editing dangerous in one specific way ([C15.6](06-editing-track.md)): because the chunks
are packed back-to-back and each expects its records 16-aligned, inserting or growing *any* chunk by a non-multiple
of 16 misaligns *every chunk after it* in the section — corrupting props, ghosting collision, and crashing on load.
The safe edit is **size-neutral** (absorb deltas into null-padding, keep every chunk's start fixed mod 16/mod 128) —
the same discipline the collision families use ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)). The
alignment invariant is the thread that binds the whole bundle: it's not per-chunk, it's per-section.

> 🟡 *Reasoned:* the 16-byte record alignment holds across every record-bearing chunk in the retail section blobs
> (scenery, terrain, smackable, collision) and is enforced by the loaders' `(ptr + 0x17) & ~0xF`; the "non-16 shift
> corrupts downstream" failure mode is the observed consequence, confirmed by byte-for-byte section rebuilds. The
> 2048-section stride and the per-chunk IDs are verified.

## RE implications

- **A section is a bundle** — render geometry + textures + scenery + collision, co-located and streamed together.
- **Visual and physical are separate data** — coarse collision vs. render meshes — but **co-located** so a tile
  arrives complete.
- **One unit of residency** — the whole bundle lives and dies together as the player enters/leaves the tile.
- **The alignment invariant is per-section** — every chunk's records are 16-aligned; edits must be size-neutral or
  the whole section misaligns.

---

### Key takeaways

- A stream section is a **bundle of co-located chunks** for one world tile — `SolidList` geometry
  ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), `TPK` textures
  ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), scenery (`0x34101/2/3`,
  [Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)), and collision (terrain `0x34159`, walls `0x3B801`,
  smackables `0x34027`, FX `0x3BC00`, [Chapter 63](../C63-Collision-World/C63-Collision-World.md)).
- The section is the **unit of world residency** — everything streams in and out together, so a tile's **visuals and
  physics always arrive as a matched set**.
- The visual world (render meshes) and the physical world (coarse collision) are **separate data, co-located** — the
  game never streams a mesh without its collision or vice-versa.
- Every record-bearing chunk obeys the **16-byte alignment invariant** `(ptr + 0x17) & ~0xF`; the section is
  **2048-aligned** — so section edits must be **size-neutral** ([C15.6](06-editing-track.md)) or every downstream
  chunk misaligns.
- This page is the **reference map** of a section — the other world chapters decode each chunk in depth; this one
  shows how they sit **together** in the streamed blob.

**Continue:** [Chapter 16 — Scenery, Props & the Cull Tree](../C16-Scenery-Cull/C16-Scenery-Cull.md) ·
[Chapter 15 hub](C15-Track-Streaming.md)
