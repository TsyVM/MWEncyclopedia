# C9.5 — Shading Groups Partition the Mesh

> **The one-sentence version:** the vertex and index buffers are shared, and each shading group owns a
> contiguous **range** of them — so a solid is really a set of per-texture sub-meshes, and rendering means
> walking the groups, binding each one's texture, and drawing its slice.

[← C9.4 — The index buffer](04-index-buffer.md) · [Chapter 9 hub](C9-Meshes-FVF.md) ·
[Next: C9.6 — Assembling triangles →](06-assembling.md)

---

## One mesh, many groups

A solid has *one* vertex buffer and *one* index buffer, but it is drawn with several textures — a car body
group, window groups, trim groups ([C7.4](../C7-Materials-TexAnim/04-usage-names.md)). The bridge is the
shading-group table ([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)): each group record carries a
**range** into the shared buffers plus the texture and material to use for that range. The buffers are the raw
geometry; the groups are the cutting pattern that turns them into textured sub-meshes.

## The range fields

Each 104-byte group record ends with range fields that grow monotonically across groups — the start/count
signature ([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)). Concretely, a group defines a contiguous
run of triangles (equivalently, a run of indices) that it owns, and the runs partition the index buffer with
no gaps and no overlap: group 0 draws triangles `[0, n₀)`, group 1 draws `[n₀, n₀+n₁)`, and so on, until every
triangle belongs to exactly one group. That total-coverage, no-overlap property is what makes the partition a
clean cut rather than an arbitrary selection.

```python
def group_triangle_ranges(groups):
    start = 0
    for g in groups:
        count = g.tri_count                 # from the group's range fields
        yield (g, start, start + count)     # triangles [start, start+count)
        start += count
    # start should now equal the solid's total triangle count
```

The check that `Σ group.tri_count == header.num_tris` is the partition's correctness test: if the group
counts don't sum to the object's triangle count, either a range field was misread or the mesh is malformed.

## Group-relative vs absolute indexing

Two indexing conventions appear in mesh formats, and it matters which one a group uses:

- **Absolute** — a group's indices point directly into the whole vertex buffer.
- **Group-relative** — a group's indices are offset by a per-group vertex base, so index `i` in the group
  means vertex `base + i`.

MW's grouping carries per-group vertex/index bases in the range fields precisely so groups can be addressed
independently. When you extract a single group as a standalone mesh you must resolve to absolute vertex
indices (add the base); when you draw in place, the engine applies the base for you. The practical rule:
**always resolve to absolute indices before exporting a group on its own**, or its triangles reference the
wrong vertices.

> 🟡 *Reasoned (indexing detail):* that groups carry per-group bases (enabling group-relative addressing) is
> inferred from the monotone range fields and the standard MW mesh design; the ✅ verified facts are that
> groups exist, are 104 bytes, and carry monotone start/count range fields that partition the buffer
> ([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)). Confirm the base convention on a multi-group solid
> before extracting a single group.

## Why partition at all

The group partition serves three systems at once:

- **Texturing.** Each group binds one texture, so the partition is *by material* — it is how one mesh shows
  many textures ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)).
- **Culling.** Each group has its own bounding box ([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)), so
  off-screen groups are skipped even when the solid is partly visible.
- **Draw batching.** Contiguous ranges mean each group is a single draw call over a slice of the buffer — no
  per-triangle overhead.

## Editing implications

- **Edits within a group are safest.** Recoloring or retexturing a group changes its record, not the ranges.
- **Adding/removing triangles shifts ranges.** Insert triangles for group *k* and every later group's start
  moves by the delta; rewrite the range fields or groups will draw the wrong triangles
  ([C8.7](../C8-Geometry-Solids/07-editing.md)).
- **Keep the partition total and disjoint.** After any edit, `Σ group.tri_count` must still equal the header
  triangle count, and ranges must not overlap.
- **Resolve bases when extracting.** Pull a group out as its own mesh only after converting to absolute vertex
  indices.

---

### Key takeaways

- A solid shares one vertex and one index buffer; shading groups own contiguous **ranges** of them.
- Ranges partition the index buffer with full coverage and no overlap — one group per triangle.
- `Σ group.tri_count == header.num_tris` is the partition's correctness check.
- Groups may address vertices relative to a per-group base; resolve to absolute indices before extracting a
  group alone.
- The partition serves texturing, culling, and draw batching simultaneously; edits must keep ranges
  consistent.

**Continue:** [C9.6 — Assembling triangles](06-assembling.md) · [Chapter 9 hub](C9-Meshes-FVF.md)
