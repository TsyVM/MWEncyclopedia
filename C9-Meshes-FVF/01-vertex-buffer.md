# C9.1 — The Vertex Buffer

> **The one-sentence version:** chunk `0x00134B01` is a packed array of fixed-size vertices behind an MW fill
> marker; the one number you must get right is the **stride** (bytes per vertex), and the unit-length normal
> at a fixed offset is the test that tells you you've found it.

[← Chapter 9 hub](C9-Meshes-FVF.md) · [Next: C9.2 — The 36-byte vertex, decoded →](02-vertex-decoded.md)

---

## The chunk

The vertex buffer is `0x00134B01` inside the mesh container ([C7.1](../C7-Materials-TexAnim/01-mesh-container.md)).
Like the index buffer, it opens with a run of `0x11` fill bytes (MW's alignment/sentinel marker), after which
the vertex data proper begins. For `COBALTSS_BASE_A` the chunk is 51 848 bytes.

The buffer is a flat, tightly packed array: vertex 0, then vertex 1, then vertex 2, each exactly `stride`
bytes, no separators. There is no per-vertex header and no padding between vertices — the format is uniform
across the whole buffer. So decoding is trivial *once you know the stride and where the data starts*, and
finding those two facts is the entire game.

## Finding the data start and stride

The 0x11 marker region is not always a tidy 8 bytes for the vertex buffer — in the worked car the vertex data
begins a little further in, after the marker and a short buffer preamble. Rather than assume a fixed offset,
**detect** it, using a property real vertices have and noise does not: a **unit-length normal** at a fixed
position within each vertex.

```python
import math, struct

def is_unit(v3, tol=0.1):
    m = math.sqrt(sum(c*c for c in v3))
    return abs(m - 1.0) < tol

def detect_layout(vb, strides=(60, 36, 24), normal_at=12):
    for start in range(8, 128, 4):
        for st in strides:
            hits = 0
            for k in range(12):
                o = start + k*st
                if o + 24 > len(vb): break
                pos = struct.unpack_from("<3f", vb, o)
                nrm = struct.unpack_from("<3f", vb, o + normal_at)
                if all(-1000 < c < 1000 for c in pos) and is_unit(nrm):
                    hits += 1
            if hits >= 11:            # 11 of 12 consecutive vertices agree
                return start, st
    raise ValueError("could not detect vertex layout")
```

The logic is robust because both conditions must hold for *many consecutive* candidate vertices: the position
triple must be finite and modest, and the normal triple must be unit length. A wrong stride or start scatters
the "normals" into non-unit garbage almost immediately, so a run of 11–12 agreeing vertices is decisive. Run
against the worked car, it returns **stride 36** with the vertex data starting after the marker/preamble, and
the first decoded vertices have positions inside the object's bounding box and unit normals — two independent
confirmations ([C9.2](02-vertex-decoded.md)).

## Why the stride is everything

Every field's offset is relative to the vertex start and repeats every `stride` bytes. Choose the wrong
stride and:

- positions drift into implausible magnitudes,
- "normals" stop being unit length,
- UVs land on random floats,
- and the vertex count `(bufferBytes − headerBytes) / stride` comes out non-integer.

That last point is a useful cross-check: the correct stride divides the data region evenly into a whole
number of vertices. For the worked car, the data region divided by 36 yields the vertex count that also makes
the index buffer's triangle indices in range — the buffers agree only under the right stride.

## Vertex count

Once you have start and stride, the vertex count is `(len(vb) − start) / stride`. This count bounds the index
values: every `u16` index in the index buffer ([C9.4](04-index-buffer.md)) must be less than the vertex
count, which is the final proof that vertex and index buffers were decoded consistently — if an index exceeds
the vertex count, either the stride or the start is wrong.

> ✅ *Verified:* the vertex buffer is a uniform packed array; stride 36 is detected by the unit-normal test on
> ~1,000 car solids, with positions inside each object's bbox and vertex counts that keep all indices in
> range.

---

### Key takeaways

- `0x00134B01` is a tightly packed array of fixed-`stride` vertices behind a `0x11` fill marker.
- Detect the data start and stride with the **unit-normal test** — real vertices have unit normals at a fixed
  offset; noise does not.
- The correct stride divides the data region into a whole number of vertices and keeps all indices in range.
- Vertex count = `(bufferBytes − headerBytes) / stride`; it bounds every index value.
- Get the stride right and every field falls into place; get it wrong and the buffer is noise.

**Continue:** [C9.2 — The 36-byte vertex, decoded](02-vertex-decoded.md) · [Chapter 9 hub](C9-Meshes-FVF.md)
