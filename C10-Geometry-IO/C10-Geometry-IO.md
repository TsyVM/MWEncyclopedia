# Chapter 10 — Geometry Import/Export & Mesh Rebuilding

> **Goal of this chapter:** move a solid out of Most Wanted into a standard 3D format (OBJ or glTF) for
> editing, and bring edited geometry back in — correctly crossing the Z-up↔Y-up boundary, preserving what
> those formats can't natively carry, and rebuilding the vertex/index buffers with all size-tree bookkeeping
> intact.

Chapters 8 and 9 gave you a solid as an in-memory mesh: vertices, triangles, groups, textures. This chapter
is the round-trip — the practical craft of getting that mesh into a tool an artist can use and back into a
form the game will load. It is where the coordinate system, the vertex format, the group partition, and the
size tree all have to be honoured at once, because a mesh that ignores any one of them either looks wrong or
won't load.

> **Built on verified structure.** Nothing here invents new file layout; it composes the verified decoders of
> [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md) (object header, Z-up bbox) and
> [Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md) (stride-36 vertices, `u16` triangle list, group ranges) into
> an export/import pipeline, and applies the size-tree discipline of
> [Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md).

---

## Deep-dive pages

- [C10.1 — The coordinate boundary: Z-up ↔ Y-up](01-coordinate-boundary.md): the one conversion that must
  happen exactly once, in one place, on export and its inverse on import.
- [C10.2 — Exporting to OBJ](02-obj-export.md): writing vertices, UVs, normals, and per-material triangle
  groups, plus the companion `.mtl`.
- [C10.3 — Exporting to glTF](03-gltf-export.md): a richer target that carries normals, multiple materials,
  and a scene graph faithfully.
- [C10.4 — What OBJ/glTF can't carry](04-lossy-boundaries.md): vertex colors, exact packed normals, asset
  keys, group bases — the side-channel you must keep.
- [C10.5 — Re-importing & rebuilding buffers](05-reimport-rebuild.md): parsing edited geometry back into
  stride-36 vertices and a `u16` index buffer.
- [C10.6 — Size-tree consequences & verification](06-sizetree-verify.md): propagating the new buffer sizes and
  proving the rebuilt solid loads.

---

## 10.1 One conversion, one place

Most Wanted is **Z-up** ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)); OBJ, glTF, Blender, and Maya are
**Y-up**. So export must rotate the world once — the standard mapping sends engine `(x, y, z)` to tool
`(x, z, −y)` (a −90° rotation about X) — and import must apply the exact inverse. The discipline that keeps
this sane is to do the conversion **only at the boundary**: keep every in-engine structure in native Z-up, and
convert solely as bytes leave for a tool and as they return. Convert in the middle and axis confusion leaks
into transforms and boxes ([C10.1](01-coordinate-boundary.md)).

## 10.2 OBJ: simple and universal

OBJ is the lowest-common-denominator target: `v` positions, `vt` UVs, `vn` normals, and `f` faces grouped by
material via `usemtl`, with a companion `.mtl` naming each material. It maps cleanly onto a solid — vertices
become `v`/`vt`/`vn`, and each shading group ([C9.5](../C9-Meshes-FVF/05-group-ranges.md)) becomes a `usemtl`
block of faces. Its limits (no vertex color, single UV set, no scene graph) are exactly the things you keep in
a side-channel ([C10.4](04-lossy-boundaries.md)). Full writer: [C10.2](02-obj-export.md).

## 10.3 glTF: richer fidelity

When you need normals, multiple materials, colors, and structure preserved faithfully, glTF is the better
target: it carries per-vertex attributes (including color), multiple primitives (one per material group), and
a scene graph, so a solid round-trips with far less lost. It is more work to write than OBJ but avoids most of
the side-channel juggling ([C10.3](03-gltf-export.md)).

## 10.4 Mind the lossy edges

Neither format natively represents everything a solid carries. The four things to preserve out-of-band are
**vertex colors** (OBJ has none), **the exact packed normal/tangent encoding** (tools re-derive normals),
**texture asset keys** ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)) (materials come back by name,
not key), and **group vertex bases** ([C9.5](../C9-Meshes-FVF/05-group-ranges.md)). Keep a sidecar mapping so
re-import can restore them ([C10.4](04-lossy-boundaries.md)).

## 10.5 Rebuilding buffers

Import is export in reverse plus a rebuild: parse the tool's vertices/faces, convert Y-up→Z-up, re-pack each
vertex into the solid's **exact stride** (36 for cars: position + normal + color + UV —
[C9.2](../C9-Meshes-FVF/02-vertex-decoded.md)), and rebuild the `u16` triangle-list index buffer with the
group ranges re-derived from the `usemtl`/primitive boundaries. The rebuilt buffers must satisfy the same
three identities that validated the original ([C9.6](../C9-Meshes-FVF/06-assembling.md)). Details:
[C10.5](05-reimport-rebuild.md).

## 10.6 Then fix the sizes

A rebuilt mesh almost never has the original's exact byte sizes, so import ends where all MW editing ends: the
**size tree**. The new vertex and index buffer sizes propagate up through `0x80134100`, `0x80134010`,
`0x80134000`, the object directory offsets ([C8.7](../C8-Geometry-Solids/07-editing.md)), and the containing
bundle. Verify with the five-point re-parse before shipping the file ([C10.6](06-sizetree-verify.md)).

---

### Key takeaways

- Export/import crosses **Z-up ↔ Y-up**; do the conversion exactly once, at the boundary, never internally.
- OBJ is universal but lossy (no color, one UV, no graph); glTF preserves normals, colors, materials, and
  structure.
- Keep a side-channel for vertex colors, exact normals/tangents, texture **asset keys**, and group bases.
- Re-import rebuilds vertices at the solid's exact stride and a `u16` triangle-list index buffer, satisfying
  the three assembly identities.
- Finish with size-tree fixups (buffers → containers → directory → bundle) and the five-point verification.

**Next:** [Chapter 11 — Attribute Vaults: VPAK Structure](../C11-Attribute-Vaults/C11-Attribute-Vaults.md):
leaving geometry for the data-driven side of the engine.
