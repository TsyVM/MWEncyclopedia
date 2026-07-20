# C9.2 — The 36-Byte Vertex, Decoded

> **The one-sentence version:** a car vertex is 36 bytes — position (3×f32), a unit normal (3×f32), a 4-byte
> packed color, and a UV pair (2×f32) — verified because the normals come out unit length and the positions
> land inside the object's bounding box.

[← C9.1 — The vertex buffer](01-vertex-buffer.md) · [Chapter 9 hub](C9-Meshes-FVF.md) ·
[Next: C9.3 — The FVF system →](03-fvf-strides.md)

---

## The layout

| Offset | Size | Field | Verified property |
|---|---|---|---|
| `+0x00` | 3 × f32 | **position** (x, y, z) | inside the header bbox, in Z-up local space |
| `+0x0C` | 3 × f32 | **normal** (x, y, z) | magnitude ≈ 1.0 |
| `+0x18` | 4 × u8 | **color** (packed) | commonly `0xFFFFFFFF` |
| `+0x1C` | 2 × f32 | **UV** (u, v) | texture coordinates |

Three worked vertices from `COBALTSS_BASE_A`:

```
pos=(-0.936, 0.693, 0.921)  nrm=(-0.044, 0.927, 0.373)  |nrm|=1.00
pos=(-0.650, 0.699, 0.929)  nrm=(-0.029, 0.925, 0.380)  |nrm|=1.00
pos=(-0.651, 0.716, 0.882)  nrm=(-0.028, 0.937, 0.347)  |nrm|=1.00
```

Every position sits inside the object's box (min (−2.20, −0.798, 0.530), max (+1.27, +0.798, 1.222)), and
every normal is unit length to two decimals — the two facts that prove the field offsets are right.

## Position

Three 32-bit floats, in the object's **local Z-up space** ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)).
They are the raw model coordinates; to place the vertex in the world you apply the object's transform
([C8.5](../C8-Geometry-Solids/05-transform.md)) and any parent hierarchy. For a car base (identity transform)
local *is* car space. Positions are bounded by the header box, which is how the culler and the loader
pre-size things — and why the box must be recomputed if you move vertices ([C8.7](../C8-Geometry-Solids/07-editing.md)).

## Normal

Three 32-bit floats forming a **unit vector** — the surface normal used for lighting. Unit length is not just
a nicety; it is the property that let this book *confirm* the layout across ~1,000 solids
([C9.1](01-vertex-buffer.md)). When you edit or regenerate geometry you must re-normalize normals, or lighting
goes wrong (over-bright or dark facets). In Z-up space, a normal pointing "up" is +Z; a car roof's normals
cluster around +Z, a door's around ±Y.

## Color

Four bytes — a packed vertex color, typically `0xFFFFFFFF` (opaque white) on car body vertices, meaning "no
tint; take the texture as-is." Vertex color multiplies the sampled texture, so non-white values darken or tint
a surface per-vertex (used for baked ambient occlusion, shadowing, or team-color effects). The byte order is
the D3D packed convention (BGRA/ARGB in memory as with textures — [C6.4](../C6-Texture-Codecs/04-argb32.md));
when in doubt, `0xFFFFFFFF` is order-agnostic, and you confirm the order on a vertex that is *not* white.

## UV

Two 32-bit floats, the texture coordinates into the group's bound texture
([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)). The convention is the usual 0..1 across the texture,
with values outside that range indicating tiling/wrap. UVs are what tie a vertex to a *place* on its texture;
edit them to move art across a surface, and remember that animated surfaces scroll these coordinates rather
than change pixels ([C7.6](../C7-Materials-TexAnim/06-texture-animation.md)).

## Decoding

```python
def decode_vertex_36(v):     # v = 36-byte slice
    return {
        "pos":    struct.unpack_from("<3f", v, 0x00),
        "normal": struct.unpack_from("<3f", v, 0x0C),
        "color":  v[0x18:0x1C],                       # 4 bytes packed
        "uv":     struct.unpack_from("<2f", v, 0x1C),
    }

def decode_vertices(vb, start, count, stride=36):
    return [decode_vertex_36(vb[start+i*stride : start+i*stride+stride])
            for i in range(count)]
```

## Editing implications

- **Re-normalize normals** after any position edit that changes surface orientation, or lighting breaks.
- **Keep positions inside a recomputed box** ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)).
- **Preserve the stride and field offsets exactly** — a 36-byte vertex must stay 36 bytes for an in-place
  edit; changing the component set is a format change and a repack.
- **Watch color and UV order/convention** when importing from external tools, which may use RGBA and a
  flipped V; convert at the boundary ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)).

> ✅ *Verified:* the 36-byte layout (position, unit normal, 4-byte color, UV) reproduces unit normals and
> in-box positions on real car vertices; `0xFFFFFFFF` colors are typical of untinted body vertices.
> 🟡 *Reasoned:* the exact color byte order (BGRA vs RGBA) follows the D3D packed convention; confirm on a
> non-white vertex before relying on a specific channel order.

---

### Key takeaways

- Car vertex = 36 bytes: position (`+0x00`), unit normal (`+0x0C`), packed color (`+0x18`), UV (`+0x1C`).
- Unit normals + in-box positions are the proof the layout is correct.
- Positions are Z-up local coordinates; apply the transform for world space.
- Re-normalize normals and recompute the box after edits; keep the stride fixed for in-place edits.
- Color multiplies the texture (`0xFFFFFFFF` = untinted); UVs index the bound texture and are what animate.

**Continue:** [C9.3 — The FVF system: strides 24 / 36 / 60](03-fvf-strides.md) · [Chapter 9 hub](C9-Meshes-FVF.md)
