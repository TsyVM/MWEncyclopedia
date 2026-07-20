# C9.4 — The Index Buffer

> **The one-sentence version:** chunk `0x00134B03` is an 8-byte marker followed by `numTris` triangles, each
> three little-endian `u16` indices into the vertex buffer — so `size = 8 + numTris × 6`, and that identity
> both proves the triangle count and validates your parse.

[← C9.3 — The FVF system](03-fvf-strides.md) · [Chapter 9 hub](C9-Meshes-FVF.md) ·
[Next: C9.5 — Shading groups partition the mesh →](05-group-ranges.md)

---

## The layout

The index buffer is the simplest structure in the mesh. After an 8-byte `0x11` fill marker, it is a flat
array of triangles, each triangle three `u16` vertex indices:

```
+0     8 bytes   marker (0x11 …)
+8     u16 u16 u16   triangle 0  (indices a, b, c)
+14    u16 u16 u16   triangle 1
...
```

So one triangle is 6 bytes, and the whole buffer is `8 + numTris × 6`. For `COBALTSS_BASE_A` the buffer is
8768 bytes: `(8768 − 8) / 6 = 1460` triangles — exactly the count in the object header's `+0x14` field
([C8.2](../C8-Geometry-Solids/02-object-header.md)). The two agreeing is your primary correctness check.

## u16 means a 65 535-vertex ceiling

Indices are 16-bit, so a single mesh can address at most 65 536 vertices. Real solids stay well under that
(the worked car object has 1440), which is why the format can afford compact `u16` indices. If you ever build
geometry that would exceed the ceiling, you must split it across multiple solids — the format has no 32-bit
index mode. In practice MW's objects are small enough (a car is many solids, each a panel) that this is never
a constraint you hit by accident.

## Reading triangles

```python
def read_indices(ib, num_tris):
    tris = []
    p = 8                                   # skip 8-byte marker
    for _ in range(num_tris):
        a, b, c = struct.unpack_from("<3H", ib, p)
        tris.append((a, b, c)); p += 6
    return tris
```

Every index must be less than the vertex count ([C9.1](01-vertex-buffer.md)); an out-of-range index means the
vertex stride (and thus vertex count) was decoded wrong, not that the index buffer is bad. This cross-buffer
check is the strongest validation you have: vertex and index buffers are only mutually consistent under the
correct stride.

## Triangle list, not strip

MW's index buffer is a **triangle list** — every three indices is one independent triangle — not a triangle
strip or fan. That means no shared-edge state between consecutive triangles and no restart indices to worry
about; triangle *k* is simply `indices[3k : 3k+3]`. Winding order (clockwise vs counter-clockwise) determines
front/back facing for culling; preserve the original winding when you rebuild geometry, or faces cull
inside-out ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)).

## Winding, normals, and facing

Two things decide whether a triangle faces the camera: its **winding order** and its vertices' **normals**
([C9.2](02-vertex-decoded.md)). They should agree — the winding's implied geometric normal should point the
same way as the stored vertex normals. When importing edited geometry, if surfaces render dark or vanish, the
usual cause is flipped winding (or flipped normals) from a tool that uses the opposite convention. Keep both
consistent with the original.

## Editing implications

- **Triangle count must match everywhere.** If you add or remove triangles, update the header `+0x14`, the
  mesh descriptor's index count (`+0x2C`), and keep `8 + numTris × 6 == indexBufferSize`
  ([C7.1](../C7-Materials-TexAnim/01-mesh-container.md)).
- **Indices must stay in range** of the vertex count; removing vertices without fixing indices dangles them.
- **Preserve winding.** Don't reorder a triangle's three indices arbitrarily — it flips the face.
- **Respect group ranges.** Triangles belong to shading groups by range ([C9.5](05-group-ranges.md)); inserting
  triangles mid-buffer shifts every later group's range and must be accounted for.

> ✅ *Verified:* index buffer = 8-byte marker + `numTris × 3 × u16`; `COBALTSS_BASE_A` = `8 + 1460 × 6 = 8768`,
> matching the header triangle count; it is a triangle **list**.

---

### Key takeaways

- `0x00134B03` = 8-byte marker + `numTris` triangles × three `u16` indices; `size = 8 + numTris × 6`.
- The triangle count matches the object header `+0x14` — the primary cross-check.
- 16-bit indices cap a mesh at 65 536 vertices; MW solids stay well under.
- It is a triangle **list** (independent triangles), not a strip; winding sets facing.
- On edit: keep counts consistent across header/descriptor/buffer, indices in range, winding preserved, and
  group ranges updated.

**Continue:** [C9.5 — Shading groups partition the mesh](05-group-ranges.md) · [Chapter 9 hub](C9-Meshes-FVF.md)
