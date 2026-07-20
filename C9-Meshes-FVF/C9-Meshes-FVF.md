# Chapter 9 — Meshes, FVF & Vertex Formats

> **Goal of this chapter:** turn a solid's two raw buffers into triangles you can render or export — decode
> the vertex buffer given its stride, read the index buffer as triangles, and understand how the shading
> groups partition both so a solid becomes a set of textured, shaded surfaces.

Chapter 8 gave you the object: its header, box, and place in the world. This chapter opens the mesh
container's two buffers and reconstructs geometry from them. The vertex buffer is a packed array of vertices
whose layout depends on a **flexible vertex format (FVF)**; the index buffer is a list of triangles; and the
shading groups ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) say which triangles use which
texture. Put together, they are a renderable mesh.

> **Verified against retail data.** The vertex layout below is decoded from `CARS/COBALTSS/GEOMETRY.BIN` and
> cross-checked across **~1,000 car solids** (COBALTSS, WHEELS, TRAILERB): stride **36**, positions inside
> each object's bounding box, and **unit-length normals** at a fixed offset — the strongest signal that the
> layout is read correctly. Index buffers decode as `numTris × 3 × u16` (e.g. `COBALTSS_BASE_A`: 1460
> triangles = 8768-byte buffer − 8-byte marker, ÷ 6).

---

## Deep-dive pages

- [C9.1 — The vertex buffer](01-vertex-buffer.md): chunk `0x00134B01`, its marker, where vertex data begins,
  and how the stride is the key to everything.
- [C9.2 — The 36-byte vertex, decoded](02-vertex-decoded.md): position, unit normal, packed color, and UV —
  field by field, verified.
- [C9.3 — The FVF system: strides 24 / 36 / 60](03-fvf-strides.md): why one stride isn't enough, what each
  component set contains, and how to detect the stride.
- [C9.4 — The index buffer](04-index-buffer.md): chunk `0x00134B03`, the 8-byte marker, and `u16` triangles.
- [C9.5 — Shading groups partition the mesh](05-group-ranges.md): how each group's range carves the shared
  buffers into per-texture sub-meshes.
- [C9.6 — Assembling triangles](06-assembling.md): the full path from buffers + groups to a list of textured
  triangles, with the correctness checks that prove it.

---

## 9.1 Two buffers, one stride

The mesh container ([C7.1](../C7-Materials-TexAnim/01-mesh-container.md)) holds a **vertex buffer**
(`0x00134B01`) and an **index buffer** (`0x00134B03`), each opening with an MW fill marker (a run of `0x11`
bytes). The index buffer is simple — after its 8-byte marker it is `numTris` triangles of three `u16`
indices each. The vertex buffer is where the format lives: it is a packed array of fixed-size vertices, and
the single most important number is the **stride** — the byte size of one vertex. Get the stride right and
every field falls into place; get it wrong and the whole buffer is noise ([C9.1](01-vertex-buffer.md)).

## 9.2 The verified 36-byte vertex

For car geometry the stride is **36 bytes**, laid out as:

| Offset | Size | Field | Notes |
|---|---|---|---|
| `+0x00` | 3 × f32 | **position** (x, y, z) | in the object's Z-up local space, inside its bbox |
| `+0x0C` | 3 × f32 | **normal** (x, y, z) | **unit length** — the verification anchor |
| `+0x18` | 4 × u8 | **color** (RGBA/BGRA) | often `0xFFFFFFFF` (white) |
| `+0x1C` | 2 × f32 | **UV** (u, v) | texture coordinates |

The unit-length normal is what makes this decode *provable*: across ~1,000 solids the three floats at `+0x0C`
have magnitude ≈ 1, which a random misalignment could not produce. Positions lie inside the header's bounding
box ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)), the second independent check. Field detail:
[C9.2](02-vertex-decoded.md).

## 9.3 Why "flexible" — the 24 / 36 / 60 family

Not every surface needs the same vertex data, so MW uses a **flexible vertex format**: different meshes carry
different component sets, and the stride reflects which. Three strides recur — **24, 36, and 60 bytes** — for
progressively richer vertices (from bare position + normal, through the 36-byte position/normal/color/UV
verified above, up to 60-byte vertices that add a tangent basis for normal mapping). Because the stride is
what distinguishes them, decoding an unfamiliar buffer starts by *determining the stride*, and the
unit-normal test is the reliable detector ([C9.3](03-fvf-strides.md)).

## 9.4 Indices and triangles

The index buffer ([C9.4](04-index-buffer.md)) is the easy half: 8-byte marker, then `numTris` triangles,
each three little-endian `u16` vertex indices into the vertex buffer. `COBALTSS_BASE_A`'s 8768-byte buffer is
exactly `8 + 1460 × 6`, so `numTris = 1460` — the same count the header stores at `+0x14`
([C8.2](../C8-Geometry-Solids/02-object-header.md)), each confirming the other.

## 9.5 Groups make sub-meshes

A solid is not one texture — it is partitioned into shading groups
([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)), each owning a contiguous **range** of the index
buffer and binding one texture. Rendering walks the groups: for each, bind its texture, draw its index range.
So "the mesh" is really a set of per-texture sub-meshes sharing one vertex and one index buffer, sliced by
group ranges ([C9.5](05-group-ranges.md)). This is why you decode buffers *and* groups to reconstruct what
the solid actually looks like.

---

### Key takeaways

- A mesh is two buffers: vertex (`0x00134B01`) and index (`0x00134B03`), each after an 8-byte `0x11` marker.
- The vertex **stride** is the master key; car vertices are 36 bytes = position + unit normal + color + UV
  (verified across ~1,000 solids).
- MW uses a **flexible vertex format** with strides 24 / 36 / 60 for different component sets; detect the
  stride via the unit-normal test.
- The index buffer is `numTris × 3 × u16`; the count matches the header's `+0x14`.
- Shading groups partition the shared buffers into per-texture sub-meshes by index range.

**Next:** [Chapter 10 — Geometry Import/Export & Mesh Rebuilding](../C10-Geometry-IO/C10-Geometry-IO.md):
taking these meshes out to OBJ/glTF and back across the Z-up↔Y-up boundary.
