# C10.5 — Re-importing & Rebuilding Buffers

> **The one-sentence version:** import is export in reverse plus a rebuild — parse the edited mesh, convert
> Y-up→Z-up, re-pack every vertex into the solid's exact stride, rebuild the `u16` triangle-list index buffer,
> and re-derive group ranges from the sidecar — producing buffers that pass the same three assembly identities
> the original did.

[← C10.4 — What OBJ/glTF can't carry](04-lossy-boundaries.md) · [Chapter 10 hub](C10-Geometry-IO.md) ·
[Next: C10.6 — Size-tree consequences & verification →](06-sizetree-verify.md)

---

## The rebuild pipeline

```python
def rebuild_solid(edited_mesh, sidecar):
    stride = sidecar["stride"]                         # e.g. 36 (C10.4)
    # 1. vertices: Y-up → Z-up, re-pack to the exact stride (C9.2, C10.1)
    vb = bytearray(b"\x11" * 8)                        # marker + preamble as original
    for v in edited_mesh.vertices:
        pos = y_up_to_z_up(v.pos)
        nrm = normalize(y_up_to_z_up(v.normal))
        col = v.color if v.has_color else b"\xff\xff\xff\xff"
        uv  = (v.uv[0], 1.0 - v.uv[1])                 # invert the export V-flip
        vb += pack_vertex(stride, pos, nrm, col, uv)   # 36: 3f+3f+4B+2f
    # 2. indices: u16 triangle list, winding preserved (C9.4)
    ib = bytearray(b"\x11" * 8)
    for (a, b, c) in edited_mesh.triangles:
        ib += struct.pack("<3H", a, b, c)
    # 3. groups: re-derive ranges from sidecar bases/counts (C9.5)
    groups = rebuild_groups(edited_mesh, sidecar["groups"])
    return vb, ib, groups
```

Three parts, each the inverse of an export step, each with a trap to avoid.

## Vertices: re-pack to the exact stride

The imported vertices must be written in the **same stride and field order** the solid used
([C9.2](../C9-Meshes-FVF/02-vertex-decoded.md)) — 36 bytes of position + normal + color + UV for a car, taken
from the sidecar so you never guess. Watch:

- **Convert Y-up→Z-up** on positions and normals ([C10.1](01-coordinate-boundary.md)), and **re-normalize**
  normals (edits and tool round-trips denormalize them).
- **Restore color** from the sidecar or glTF `COLOR_0`; default to `0xFFFFFFFF` only when the original was
  white ([C10.4](04-lossy-boundaries.md)).
- **Invert the UV V-flip** you applied on export so texture coordinates match MW's convention again.
- **Match the marker/preamble** the original buffer had before the vertex data
  ([C9.1](../C9-Meshes-FVF/01-vertex-buffer.md)), so offsets line up.

## Indices: rebuild the triangle list

Write triangles as three little-endian `u16` each, in the **original winding order**
([C9.4](../C9-Meshes-FVF/04-index-buffer.md)). Two constraints:

- **Vertex count ≤ 65 536.** If editing added vertices past the `u16` ceiling, you must split the solid — the
  format has no 32-bit index mode.
- **Indices in range.** Every index must be `< vertexCount`; a tool that re-ordered or merged vertices can
  leave dangling indices, so validate before writing.

## Groups: reconstruct the partition

The `usemtl`/primitive boundaries in the edited file tell you which triangles belong to which material; the
sidecar tells you each group's **vertex base**, **texture asset key**, and material parameters
([C10.4](04-lossy-boundaries.md)). Rebuild each 104-byte group record
([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)) with:

- its **triangle range** (start/count) into the rebuilt index buffer, in group order;
- its **bounding box**, recomputed from the group's vertices in Z-up space
  ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md));
- its **texture reference**, restored from the sidecar's asset key
  ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)).

Keep the group **count** in the mesh descriptor (`+0x10`) in sync
([C7.1](../C7-Materials-TexAnim/01-mesh-container.md)), and make sure `Σ group.tri_count` equals the new
triangle total.

## The three identities must hold again

Before you write anything, re-run the assembly checks ([C9.6](../C9-Meshes-FVF/06-assembling.md)) on your
rebuilt buffers:

1. `num_tris == (len(ib) − 8) / 6`
2. every index `< (len(vb) − start) / stride`
3. `Σ group.tri_count == num_tris`, ranges disjoint and covering

If all three pass, the rebuilt mesh is internally consistent. Only then do you fix the counts in the header
and descriptor and move on to the size tree ([C10.6](06-sizetree-verify.md)). Passing these identities is what
separates a mesh that loads from one that crashes the game.

---

### Key takeaways

- Import = parse edited mesh → Y-up→Z-up → re-pack to exact stride → rebuild `u16` index list → reconstruct
  groups from the sidecar.
- Re-normalize normals, restore color, invert the UV flip, and match the buffer marker/preamble.
- Keep vertex count ≤ 65 536 and all indices in range; preserve winding.
- Rebuild each 104-byte group with its triangle range, recomputed Z-up bbox, and sidecar asset key; keep the
  group count synced.
- Re-verify the three assembly identities *before* writing — that's the load-or-crash gate.

**Continue:** [C10.6 — Size-tree consequences & verification](06-sizetree-verify.md) · [Chapter 10 hub](C10-Geometry-IO.md)
