# C10.4 — What OBJ/glTF Can't Carry

> **The one-sentence version:** interchange formats lose four MW-specific things — vertex colors (OBJ),
> the exact packed normal/tangent encoding, the texture **asset key**, and per-group vertex bases — so keep a
> small sidecar file that maps them back, and a round-trip stays faithful.

[← C10.3 — Exporting to glTF](03-gltf-export.md) · [Chapter 10 hub](C10-Geometry-IO.md) ·
[Next: C10.5 — Re-importing & rebuilding buffers →](05-reimport-rebuild.md)

---

## The four losses

| MW data | OBJ | glTF | Why it matters |
|---|---|---|---|
| **Vertex color** ([C9.2](../C9-Meshes-FVF/02-vertex-decoded.md)) | ✗ none | ✓ `COLOR_0` | baked AO/tint; wrong = flat or mis-shaded |
| **Exact normal/tangent encoding** ([C9.3](../C9-Meshes-FVF/03-fvf-strides.md)) | re-derived | re-derived | stride-60 tangent frames won't round-trip byte-exact |
| **Texture asset key** ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)) | ✗ (name only) | ✗ (image file) | the binding is by key; a name won't re-bind |
| **Per-group vertex base** ([C9.5](../C9-Meshes-FVF/05-group-ranges.md)) | ✗ | ✗ | needed to reconstruct group ranges exactly |

OBJ loses all four; glTF recovers color natively but still loses the asset key, the exact encoding, and the
group base. None of these is representable in the standard format, so the fix is not a better exporter — it is
a **sidecar**.

## The sidecar

Write a small companion file (JSON is ideal) alongside the OBJ/glTF that records, per material and per solid,
exactly what the interchange format cannot:

```json
{
  "solid": "COBALTSS_BASE_A",
  "name_hash": "0x54DF8EF4",
  "stride": 36,
  "bbox_min": [-2.20, -0.798, 0.530],
  "bbox_max": [ 1.27,  0.798, 1.222],
  "groups": [
    { "material": "BODY_ALUMINUM",  "texture_key": "0x1A2B3C4D", "vertex_base": 0,    "tri_count": 254 },
    { "material": "WINDOW_FRONT",   "texture_key": "0x0CD55E13", "vertex_base": 254,  "tri_count": 96 }
  ]
}
```

This sidecar is the authoritative record for re-import. It carries the **asset keys** so materials re-bind by
key not name ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)), the **stride** so vertices re-pack in the
right format ([C10.5](05-reimport-rebuild.md)), the **group bases and counts** so ranges reconstruct exactly,
and the original **bbox/hash** for the header. Keep it version-matched to the exported mesh (a hash or
timestamp) so you never re-import against the wrong sidecar.

## Vertex color specifically

If you must use OBJ (no `COLOR_0`), you have three options for color, in order of preference:

1. **Carry it in the sidecar** as a per-vertex array — exact, but bulky.
2. **Use glTF instead**, which stores it natively — the clean answer.
3. **Accept `0xFFFFFFFF`** (untinted white) if the surface was white anyway — verified typical for car body
   vertices ([C9.2](../C9-Meshes-FVF/02-vertex-decoded.md)), so for many panels this loses nothing.

Never silently drop non-white vertex colors: a surface that used vertex color for baked shadowing will render
flat and wrong if you re-import it as white.

## Normals and tangents

Tools re-derive normals from geometry, which is usually *fine* — a re-derived normal on unchanged geometry is
nearly identical to the original. The exception is the **stride-60 tangent frame** for normal mapping
([C9.3](../C9-Meshes-FVF/03-fvf-strides.md)): tangents depend on UVs and winding and must be recomputed
consistently, or normal maps light incorrectly. If you export a normal-mapped solid, either recompute tangents
deterministically on import or carry the originals in the sidecar.

## The principle

The interchange format holds the **shape**; the sidecar holds the **MW-specific bindings and encodings**.
Together they make a lossless round-trip; either alone does not. Design your pipeline so the sidecar is written
on every export and consumed on every import, and treat a missing or mismatched sidecar as a hard error rather
than guessing the lost data.

---

### Key takeaways

- Four things don't survive OBJ/glTF natively: vertex color (OBJ), exact normal/tangent encoding, texture
  **asset key**, and per-group vertex base.
- Keep a **sidecar** (JSON) with keys, stride, group bases/counts, bbox, and name-hash — the authoritative
  re-import record.
- For color: prefer glTF's `COLOR_0`, else sidecar it; only default to white when the surface truly was white.
- Re-derived normals are fine; stride-60 **tangents** must be recomputed consistently or carried.
- Shape lives in the interchange file, bindings in the sidecar — both are required for a faithful round-trip.

**Continue:** [C10.5 — Re-importing & rebuilding buffers](05-reimport-rebuild.md) · [Chapter 10 hub](C10-Geometry-IO.md)
