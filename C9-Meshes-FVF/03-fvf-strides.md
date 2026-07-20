# C9.3 — The FVF System: Strides 24 / 36 / 60

> **The one-sentence version:** MW meshes don't all carry the same vertex data, so the format is *flexible* —
> the stride tells you the component set, and the three that recur are 24 (lean), 36 (the verified
> position/normal/color/UV car vertex), and 60 (rich, adding a tangent basis for normal mapping).

[← C9.2 — The 36-byte vertex, decoded](02-vertex-decoded.md) · [Chapter 9 hub](C9-Meshes-FVF.md) ·
[Next: C9.4 — The index buffer →](04-index-buffer.md)

---

## Why flexible

A single fixed vertex format would waste space on simple meshes and lack data for complex ones. A shadow
volume needs only position; a matte prop needs position, normal, and UV; a normal-mapped car panel needs a
full tangent basis on top. So MW uses a **flexible vertex format (FVF)**: different meshes store different
component sets, and the **stride** — the byte size of one vertex — encodes which set is present. The engine
and any reader branch on the stride to know how to interpret each vertex.

## The three strides

Three strides recur across the game's geometry — **24, 36, and 60 bytes** — a lean/standard/rich ladder:

| Stride | Typical component set | Used for |
|---|---|---|
| **24** | position (12) + normal (12), *or* position (12) + normal (8, packed) + UV (…) | simple/opaque surfaces, low-detail geometry |
| **36** | position (12) + normal (12) + color (4) + UV (8) | **car body and standard textured surfaces (verified)** |
| **60** | 36's fields + a **tangent basis** (tangent + binormal) and/or a second UV set | normal-mapped, high-detail surfaces |

The 36-byte set is the one this book verified exhaustively ([C9.2](02-vertex-decoded.md)); the 24 and 60
strides are the lean and rich variants around it. The pattern is additive: richer strides keep the core
position/normal and append more per-vertex channels (extra UVs, colors, and the tangent frame that normal
mapping requires).

> ✅ *Verified:* stride 36 with position/normal/color/UV, across ~1,000 car solids.
> 🟡 *Reasoned:* the exact component breakdown of the 24- and 60-byte strides (which packing, which extra
> channels) follows the additive FVF pattern and the toolkit's documented 24/36/60 strides; confirm the
> precise field offsets of a 24- or 60-byte buffer with the unit-normal test before decoding its extra
> channels.

## Detecting the stride

Never assume — detect. The unit-normal test from [C9.1](01-vertex-buffer.md) doubles as a stride detector:
try each candidate stride and accept the one under which many consecutive vertices have a unit normal at the
normal offset and plausible positions. Because a wrong stride destroys the unit-normal property within a
vertex or two, the correct stride announces itself as a long run of agreeing vertices.

```python
def detect_stride(vb, start_range=range(8, 128, 4), strides=(60, 36, 24)):
    for start in start_range:
        for st in strides:
            if run_of_valid_vertices(vb, start, st) >= 11:
                return start, st
    return None
```

Two corroborating checks make the detection airtight:

- **Even division.** `(len(vb) − start) / stride` must be a whole number of vertices.
- **Index range.** Every `u16` in the index buffer must be `< vertexCount`; the right stride is the one whose
  vertex count keeps all indices in range ([C9.4](04-index-buffer.md)).

## Handling multiple strides in one reader

A robust mesh reader treats stride as data, not a constant:

```python
def read_mesh_vertices(vb):
    start, stride = detect_stride(vb)
    count = (len(vb) - start) // stride
    verts = []
    for i in range(count):
        v = vb[start + i*stride : start + (i+1)*stride]
        rec = {"pos": f3(v, 0), "normal": f3(v, 12)}   # always present
        if stride >= 36:
            rec["color"] = v[24:28]
            rec["uv"]    = f2(v, 28)
        if stride >= 60:
            rec["tangent_frame"] = decode_tangent(v, 36)   # normal-map data
        return_append(verts, rec)
    return verts
```

The core (position + normal) is universal; the richer fields are gated on the stride. This keeps one code
path for all three formats and degrades gracefully if you meet a stride you have not fully mapped — you still
recover positions, normals, and triangles, which is enough to see and export the shape.

## Editing across strides

- **Keep the stride you were given.** An in-place vertex edit must preserve the stride; changing the component
  set (e.g. 36 → 60 to add normal mapping) rewrites every vertex and is a repack
  ([C8.7](../C8-Geometry-Solids/07-editing.md)).
- **Match the stride on import.** When bringing geometry back from an external tool
  ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)), emit vertices in the *same* stride/format the solid
  originally used, or the buffer size and the header counts stop agreeing.
- **The tangent frame (stride 60) must be consistent.** If you regenerate normal-mapped geometry, recompute
  tangents from the UVs and positions; a stale tangent basis makes normal maps light incorrectly.

---

### Key takeaways

- MW's vertex format is flexible; the **stride** encodes the component set.
- Three strides recur: **24** (lean: position + normal), **36** (verified: + color + UV), **60** (rich: +
  tangent basis / extra channels).
- Richer strides are additive — they keep position/normal and append channels.
- Detect the stride with the unit-normal test, corroborated by even division and in-range indices.
- Preserve the stride for in-place edits; match it on import; keep tangent frames consistent for stride 60.

**Continue:** [C9.4 — The index buffer](04-index-buffer.md) · [Chapter 9 hub](C9-Meshes-FVF.md)
