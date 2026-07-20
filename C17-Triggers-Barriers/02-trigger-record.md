# C17.2 — The Trigger Record

> **The one-sentence version:** each trigger is a variable-stride record — a fixed head carrying the type, a
> coarse center/radius gate, the polygon's AABB (`+0x20`), and the vertex count (`+0x40`) — followed by the
> polygon's vertices, all inside the `0x80034147` parent that wraps `0x0003414A`.

[← C17.1 — Triggers as 2-D footprints](01-footprints.md) · [Chapter 17 hub](C17-Triggers-Barriers.md) ·
[Next: C17.3 — The 15 trigger types →](03-trigger-types.md)

---

## The container

Trigger regions live in `0x0003414A TriggerRegions`, wrapped by the `0x80034147 TriggerRegionParent`
container ([C15.1](../C15-Track-Streaming/01-master-layout.md)). Unlike the fixed-stride tables elsewhere in
the world, trigger records are **variable length** — because each carries its own polygon — so you cannot
divide the chunk size by a constant to count them; you walk record by record, using each record's length to
find the next.

## The record head — byte-verified

Decoded from the first real trigger in `L2RA.BUN`:

| Offset | Type | Value (record 0) | Field |
|---|---|---|---|
| `+0x00` | `u32` | `3` | **trigger type** (1–14) |
| `+0x04` | `f32` | `4337.0` | coarse center X |
| `+0x08` | `f32` | `1509.0` | coarse center Y |
| `+0x0C` | `f32` | `1.0` | (flags / height) |
| `+0x14` | `f32` | `30.0` | coarse **radius** |
| `+0x20` | `4×f32` | `(4296.4, 1436.6, 4374.8, 1584.7)` | **AABB** (minX, minZ, maxX, maxZ) |
| `+0x40` | `u16` | `8` | **vertex count** |
| `+0x42` | `u16` | — | length word |
| `≥ +0x44` | `f32[2]×n` | — | polygon vertices (X, Y pairs) |

Every one of these fields was read straight from the retail bytes: type `3`, an 8-vertex polygon, and an AABB
that is a sensible ground-plane rectangle. The base of the record is at least `0x44` bytes; the polygon
vertices follow, and the length word lets the parser advance to the next record.

## Two bounding volumes, two jobs

The record carries *two* spatial descriptions, and they are not the same thing:

- **The coarse gate** at `+0x04` — a center `(x, y)` and a radius. This is a cheap circular pre-test: if the
  car is farther than the radius from the center, skip this trigger entirely. It **does not tightly bound the
  polygon**; it's a fast reject.
- **The AABB** at `+0x20` — the polygon's tight rectangular bound, `(minX, minZ, maxX, maxZ)`. This is the
  exact hull of the vertices, used as the precise box test before the even–odd polygon test
  ([C17.4](04-even-odd.md)).

Confusing the two — treating the center/radius as the polygon's bound — is a classic error; the radius is a
loose gate, the AABB is the tight one.

## Walking the records

```python
def walk_triggers(chunk):
    triggers, off = [], 0
    while off < len(chunk):
        t = {
            "type":     u32(chunk, off + 0x00),
            "gate":     (f32(chunk, off+0x04), f32(chunk, off+0x08), f32(chunk, off+0x14)),  # cx,cy,r
            "aabb":     struct.unpack_from("<4f", chunk, off + 0x20),   # minX,minZ,maxX,maxZ
            "n_verts":  u16(chunk, off + 0x40),
        }
        base = 0x44
        t["poly"] = [struct.unpack_from("<2f", chunk, off + base + i*8) for i in range(t["n_verts"])]
        triggers.append(t)
        off += record_length(chunk, off)      # from the length word / vertex count
    return triggers
```

The one subtlety is `record_length` — you advance by the head size plus `n_verts × 8` (two floats per vertex),
respecting the length word at `+0x42`. Get it right and the walk lands on each successive type field; get it
wrong and every record after the first is misread — the variable-stride version of the wrong-stride failure
mode ([C9.1](../C9-Meshes-FVF/01-vertex-buffer.md)).

> ✅ *Verified:* the head fields (type `+0x00`, AABB `+0x20`, vertex count `+0x40`) decode correctly on the
> retail track (type 3, AABB a ground rectangle, 8 vertices). The fixed header is exactly **68 bytes** (`0x44`), and
> the word at `+0x42` is a **self-validating `recordSize = 68 + vertexCount × 8`** — checked against the actual
> consumption on every record, so a mismatch aborts the parse rather than desyncing ([the walk below](#walking-the-records)).
> Beyond the fields tabled above, `+0x0C`/`+0x10` are a **direction vector** and `+0x18`/`+0x1C` are two **flag
> words**, with 16 bytes of per-type extra at `+0x30`.
> 🟡 *Reasoned:* the exact bit meanings of the flag words and the `+0x30` extra block are per-type detail
> ([C17.3](03-trigger-types.md)); the 68-byte head layout, the `recordSize` self-check, and the polygon follow are
> verified.

## Editing implications

- **Reshape via the polygon, then fix the AABB.** Move or add vertices, then recompute
  `(minX, minZ, maxX, maxZ)` at `+0x20` to bound them ([C17.1](01-footprints.md)).
- **Keep the coarse gate sane.** The center/radius should still loosely cover the polygon, or the fast reject
  wrongly skips the trigger.
- **Variable length means repack.** Adding vertices grows the record, so following records shift and the
  `0x80034147` wrapper's size must be fixed ([C17.6](06-events-editing.md)) — a repack, not an in-place edit.

---

### Key takeaways

- Triggers live in `0x0003414A`, wrapped by `0x80034147`; records are **variable length** (each carries its
  polygon).
- Head: type (`+0x00`), coarse center/radius gate (`+0x04`), AABB `(minX,minZ,maxX,maxZ)` (`+0x20`), vertex
  count (`+0x40`), then vertices — all byte-verified on the retail track.
- Two volumes: a loose center/radius **gate** (fast reject) and a tight **AABB** (exact bound) — don't confuse
  them.
- Walk records by advancing head + `n_verts × 8`; a wrong length desyncs every later record.
- Reshape via the polygon and recompute the AABB; growing a record is a repack with wrapper-size fixups.

**Continue:** [C17.3 — The 15 trigger types](03-trigger-types.md) · [Chapter 17 hub](C17-Triggers-Barriers.md)
