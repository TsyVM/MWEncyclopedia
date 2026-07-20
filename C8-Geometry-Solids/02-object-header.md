# C8.2 — The Object Header, Field by Field

> **The one-sentence version:** each solid opens with a 176-byte header (`0x00134011`) laying out flags, the
> asset name-hash, the triangle count, a min/max bounding box, a 4×4 placement matrix, and the object's ASCII
> name — every field of which this book decoded and cross-checked against `COBALTSS_BASE_A`.

[← C8.1 — The SolidList container](01-solidlist-container.md) · [Chapter 8 hub](C8-Geometry-Solids.md) ·
[Next: C8.3 — Object names & the asset hash →](03-object-hash.md)

---

## The full layout

Dumping `COBALTSS_BASE_A`'s header (`0x00134011`, 176 bytes) and interpreting each word:

| Offset | Type | Value | Field |
|---|---|---|---|
| `+0x00` | u32×3 | 0, 0, 0 | reserved |
| `+0x0C` | u32 | `0x00400016` | flags / version |
| `+0x10` | u32 | `0x54DF8EF4` | **name-hash** (asset hash) |
| `+0x14` | u32 | `0x000005B4` = 1460 | **num triangles** |
| `+0x18` | u32 | `0x00070A00` | packed field (stream/vertex info) |
| `+0x1C` | u32 | 0 | reserved |
| `+0x20` | f32×3 | (−2.20, −0.798, 0.530) | **bbox min** (x, y, z) |
| `+0x2C` | f32 | 0 | pad |
| `+0x30` | f32×3 | (+1.27, +0.798, 1.222) | **bbox max** (x, y, z) |
| `+0x3C` | f32 | 0 | pad |
| `+0x40` | f32×16 | identity | **4×4 transform** |
| `+0x88` | u32×2 | `0x000EA550` ×2 | data offsets/sizes |
| `+0xA0` | char[] | `COBALTSS_BASE_A\0` | **name** |

The header packs the three things you need to *place and identify* an object — who it is (name + hash), how
big it is (triangle count, box), and where it sits (transform) — with the actual geometry deferred to the
mesh container ([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)).

## The fields that are load-bearing

**`+0x10` name-hash.** The asset-hash id the directory and hash table key on
([C8.1](01-solidlist-container.md)). This is how the object is referenced everywhere; the ASCII name at
`+0xA0` is for humans and tools. The hash's structure is [C8.3](03-object-hash.md).

**`+0x14` triangle count.** `0x5B4` = 1460, and this is *provable*, not assumed: the index buffer
(`0x00134B03`) is 8768 bytes = `8 (marker) + 1460 × 6`, since each triangle is three `u16` indices
([C7.1](../C7-Materials-TexAnim/01-mesh-container.md)). When you meet an unfamiliar solid, cross-checking
`+0x14` against `(indexbytes − 8) / 6` tells you instantly whether you have the header aligned correctly.

**`+0x20` / `+0x30` bounding box.** Min and max corners as three floats each (with a padding float after each
to keep 16-byte alignment). For the worked car: min (−2.20, −0.798, 0.530), max (+1.27, +0.798, 1.222). The
symmetry in Y (±0.798) and the sensible car dimensions this yields confirm both the field interpretation and
the Z-up coordinate system ([C8.4](04-bounding-boxes.md)).

**`+0x40` transform.** A 4×4 matrix, identity for a car base object (the base sits at the origin with no
rotation), non-trivial for placed sub-parts and world objects ([C8.5](05-transform.md)).

**`+0xA0` name.** Null-terminated ASCII, up to 24 bytes as with textures. `COBALTSS_BASE_A`, `..._BASE_B`,
`..._BASE_C` are the first three objects — a family of body panels.

## Reading it

```python
def parse_object_header(h):     # h = 176-byte 0x00134011 payload
    return {
        "flags":     u32(h, 0x0C),
        "name_hash": u32(h, 0x10),
        "num_tris":  u32(h, 0x14),
        "bbox_min":  struct.unpack_from("<3f", h, 0x20),
        "bbox_max":  struct.unpack_from("<3f", h, 0x30),
        "transform": struct.unpack_from("<16f", h, 0x40),
        "name":      cstr(h, 0xA0),
    }
```

## Verifying an alignment

The header is a good place to catch a mis-seek, because three of its fields are mutually constraining:

1. The **name** at `+0xA0` should be printable ASCII ending in a null — if it is garbage, your header start
   is wrong.
2. The **triangle count** at `+0x14` should satisfy `num_tris × 6 + 8 == indexBufferSize`.
3. The **bounding box** floats at `+0x20`/`+0x30` should be small, sane numbers with `min ≤ max` per axis.

If all three hold, the header is correctly located and every other field can be trusted. This triple-check is
worth building into any SolidList reader — it turns a silent misalignment into an immediate, obvious failure.

> ✅ *Verified:* every tabulated field against `COBALTSS_BASE_A`: name at `+0xA0`, name-hash `0x54DF8EF4` at
> `+0x10`, triangle count 1460 at `+0x14` (confirmed by index buffer), bbox and identity transform as shown.

---

### Key takeaways

- The object header (`0x00134011`) is 176 bytes: flags (`+0x0C`), name-hash (`+0x10`), triangle count
  (`+0x14`), bbox (`+0x20`/`+0x30`), 4×4 transform (`+0x40`), name (`+0xA0`).
- The triangle count is provable from the index buffer: `num_tris × 6 + 8 = indexBufferSize`.
- The bounding box is two float-triples with padding; min ≤ max per axis.
- Validate a header by three mutually-constraining fields: printable name, tri-count vs index buffer, sane
  box — a fast mis-seek detector.

**Continue:** [C8.3 — Object names & the asset hash](03-object-hash.md) · [Chapter 8 hub](C8-Geometry-Solids.md)
