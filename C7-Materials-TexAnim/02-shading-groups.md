# C7.2 — The Shading-Group Descriptor

> **The one-sentence version:** each shading group is a fixed 104-byte record beginning with the group's own
> min/max bounding box (verified to be a sub-region of the solid's box), followed by a material marker, a
> texture reference, and the vertex/triangle range the group draws from the shared buffers.

[← C7.1 — The mesh container](01-mesh-container.md) · [Chapter 7 hub](C7-Materials-TexAnim.md) ·
[Next: C7.3 — Binding a texture by asset key →](03-texture-binding.md)

---

## The record

The shading-group table `0x00134B02` is an 8-byte header followed by `count` records of **104 bytes** each
(for `COBALTSS_BASE_A`: 8 + 12 × 104 = 1256 bytes, exact). Decoding group 0 and group 1 of that solid gives
a consistent field map:

| Offset | Type | Group 0 | Group 1 | Role |
|---|---|---|---|---|
| `+0x00` | f32×3 | (−1.073, 0.648, 0.846) | (−2.180, −0.080, 0.706) | **bbox min** (x, y, z) |
| `+0x0C` | f32×3 | (−0.640, 0.735, 1.029) | (−2.138, 0.079, 0.773) | **bbox max** (x, y, z) |
| `+0x18` | u32 | `0x05050505` | `0x09090909` | material/marker tag (replicated byte) |
| `+0x30` | u32 | `4` | `4` | shader/material type (constant here) |
| `+0x38` | u32 | `0x00004180` | `0x00014180` | texture reference (packed) |
| `+0x3C` | u32 | 19 | 60 | range field (index/vertex start) |
| `+0x40` | u32 | 18 | 34 | range field (count) |
| `+0x5C` | u32 | 54 | 102 | range field (running total) |

## The bounding box is real (and useful)

The first 24 bytes are unmistakably a per-group AABB, and you can *prove* it: the solid's overall box is
min (−2.2, −0.798, 0.53) → max (1.27, 0.798, 1.222) ([C8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), and
every group's box sits **inside** it. Group 0 occupies x ∈ [−1.07, −0.64], group 1 the thin slab
x ≈ −2.16 near the car's end. Each group is a spatial patch of the model — a body panel, a window, a light —
which is exactly what you expect when a mesh is partitioned by material and region for culling and draw
batching.

That box is not decoration: the engine uses per-group bounds to cull groups that are off-screen even when
the whole solid is visible, and you can use them to identify what a group *is* without decoding a single
triangle (a thin box at the front is a bumper or grille; a large central box is the main body).

## The material half: type, marker, and texture reference

After the box, the record carries the group's **look**:

- `+0x30` a small **type/shader selector** (`4` here) — which rendering path the group uses.
- `+0x18` a **replicated-byte marker** (`0x05050505`, `0x09090909`): MW writes these single-byte-filled
  words as tags/sentinels throughout the format (compare the `0x11111111` buffer markers). Here the byte
  differs per group and behaves like a small per-group material/tag index.
- `+0x38` the **texture reference** (`0x…4180` packed): the group's link to a texture. In MW this reference
  is resolved indirectly — through the solid's texture table — to one of the asset-keyed textures a bound
  TPK provides ([C7.3](03-texture-binding.md)). The record stores the *link*, not the pixels.

> 🟡 *Reasoned (fields identified by role):* the bbox (`+0x00`/`+0x0C`) is ✅ verified by containment in the
> solid box; the type (`+0x30`), marker (`+0x18`), texture reference (`+0x38`) and range fields
> (`+0x3C`/`+0x40`/`+0x5C`) are identified by their consistent behaviour across groups, but the exact
> semantics of the packed texture reference and the precise index-vs-vertex meaning of each range field are
> reasoned, not bit-proven. The record **size (104) and count (`descriptor +0x10`) are ✅ verified.**

## The range fields: which triangles a group owns

The three integer fields near the end grow monotonically across groups (group 0: 19/18/54; group 1:
60/34/102), the signature of **start/count/running-total** ranges into the shared index (and vertex) buffer.
Drawing the solid means, per group: seek to the group's start in the index buffer, draw its count of
triangles with its bound texture and parameters, advance. Because the ranges partition the buffer, every
triangle belongs to exactly one group — no overlap, full coverage.

## Parsing groups

```python
def parse_groups(table, count, REC=104):
    groups = []
    for i in range(count):
        r = table[8 + i*REC : 8 + (i+1)*REC]
        g = {
            "bbox_min": struct.unpack_from("<3f", r, 0x00),
            "bbox_max": struct.unpack_from("<3f", r, 0x0C),
            "marker":   u32(r, 0x18),
            "type":     u32(r, 0x30),
            "tex_ref":  u32(r, 0x38),   # resolve via texture table → asset key → TPK (C7.3)
            "range":   (u32(r, 0x3C), u32(r, 0x40), u32(r, 0x5C)),
        }
        groups.append(g)
    return groups
```

## Editing implications

- **Changing a group's texture** means changing its reference (`+0x38` / the texture table it indexes),
  never rewriting pixels in the group record.
- **The bounding box must enclose the group's actual geometry.** If you move vertices
  ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)) you should recompute each affected group's min/max,
  or the engine may wrongly cull a group whose box no longer contains it.
- **Do not resize the record.** 104 bytes is fixed; the descriptor's group count and the table size must stay
  consistent, exactly as [C7.1](01-mesh-container.md) verifies.

---

### Key takeaways

- A shading group is a 104-byte record; count comes from descriptor `+0x10` (table = 8 + count·104).
- Bytes `+0x00`/`+0x0C` are a per-group AABB — **verified** to nest inside the solid's box; use it to
  identify parts and to drive culling.
- `+0x30` type, `+0x18` marker, `+0x38` texture reference, and three monotone range fields describe the
  group's look and its slice of the buffers.
- The group stores a texture *link*, resolved indirectly to an asset-keyed TPK texture ([C7.3](03-texture-binding.md)).
- Edit references, not pixels; keep boxes enclosing their geometry; never change the 104-byte size.

**Continue:** [C7.3 — Binding a texture by asset key](03-texture-binding.md) · [Chapter 7 hub](C7-Materials-TexAnim.md)
