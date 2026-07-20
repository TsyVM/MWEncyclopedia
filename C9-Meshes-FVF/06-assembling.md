# C9.6 — Assembling Triangles

> **The one-sentence version:** put the pieces together — detect the stride and decode vertices, read the
> `u16` triangle list, split it by group range, and attach each group's texture — and you have the solid as a
> set of textured triangles, provable by three cross-checks that must all hold.

[← C9.5 — Shading groups partition the mesh](05-group-ranges.md) · [Chapter 9 hub](C9-Meshes-FVF.md) ·
[Next: Chapter 10 — Geometry Import/Export →](../C10-Geometry-IO/C10-Geometry-IO.md)

---

## The full pipeline

Everything in this chapter and Chapter 7 composes into one procedure that turns a solid into renderable,
textured geometry:

```python
def assemble_solid(solid):
    # 1. header: counts and identity (C8.2)
    hdr = parse_object_header(solid.chunk[0x00134011])

    # 2. vertices: detect stride, decode (C9.1–C9.3)
    vb = solid.mesh[0x00134B01]
    start, stride = detect_stride(vb)
    verts = decode_vertices(vb, start, (len(vb)-start)//stride, stride)

    # 3. indices: triangle list (C9.4)
    tris = read_indices(solid.mesh[0x00134B03], hdr["num_tris"])

    # 4. groups: partition + texture (C7.2, C9.5)
    groups = parse_groups(solid.mesh[0x00134B02], group_count(solid.mesh[0x00134900]))

    # 5. stitch: per group, its triangles carry its texture
    submeshes = []
    for g, lo, hi in group_triangle_ranges(groups):
        submeshes.append({
            "texture_key": resolve_texture(g),        # C7.3
            "triangles":  [tris[t] for t in range(lo, hi)],
            "usage_name": g.get("usage_name"),        # C7.4
        })
    return {"vertices": verts, "submeshes": submeshes, "bbox": (hdr["bbox_min"], hdr["bbox_max"])}
```

The output is exactly what a renderer or exporter wants: a shared vertex array plus a list of sub-meshes, each
a set of triangles with a texture. From here, drawing is "for each sub-mesh, bind texture, draw triangles,"
and exporting is "write the vertices, then the per-material triangle groups"
([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)).

## The three cross-checks that prove it

An assembly is only trustworthy if these three independent identities hold — each catches a different class of
error:

1. **Triangle count.** `header.num_tris == (indexBytes − 8) / 6`. Catches a mis-sized or mis-located index
   buffer.
2. **Index range.** `max(all indices) < vertexCount`, where `vertexCount = (vbBytes − start) / stride`.
   Catches a wrong stride or vertex start — the two buffers are only consistent under the right stride.
3. **Group partition.** `Σ group.tri_count == header.num_tris`, ranges disjoint and covering. Catches a
   misread group table or count.

If all three pass, the vertices, indices, and groups are mutually consistent, and the assembled mesh is
correct. If one fails, it tells you *which* structure to re-examine — the failing identity names the culprit.

## A worked confirmation

For `COBALTSS_BASE_A` all three hold:

- `num_tris = 1460`, and `(8768 − 8) / 6 = 1460` ✓
- `vertexCount = 1440` (stride 36), and every index `< 1440` ✓
- 12 groups whose triangle counts sum to 1460, partitioning the index buffer ✓

Three structures — header, buffers, group table — independently agree on the same geometry. That agreement,
not any single field, is what makes the decode reliable.

## From triangles to a viewer

Once assembled, the mesh is trivially renderable: positions and normals feed lighting, UVs and the resolved
texture key ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)) feed texturing, and the per-group split
means each material draws as one batch. To *view* a solid you now need only decode its texture pack
([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) for the
keys the sub-meshes reference, and you have a fully textured car panel reconstructed entirely from the file.

## Editing implications

- **Round-trip through the assembled form.** Edit the neutral `{vertices, submeshes}` representation, then
  re-encode to buffers + groups, re-running the three checks before writing
  ([C8.7](../C8-Geometry-Solids/07-editing.md)).
- **Keep the three identities true after every edit** — they are your pre-flight test that the solid will
  load.
- **Preserve keys and winding** so textures still bind ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md))
  and faces still face out ([C9.4](04-index-buffer.md)).

---

### Key takeaways

- Assembling a solid = header counts + stride-decoded vertices + `u16` triangle list + group partition +
  texture resolution.
- Output is a shared vertex array plus per-material triangle sub-meshes — ready to render or export.
- Three cross-checks prove correctness: tri-count vs index buffer, indices in vertex range, group counts sum
  to tri-count.
- All three hold for `COBALTSS_BASE_A` (1460 tris, 1440 verts, 12 groups) — independent structures agreeing.
- Edit the neutral assembled form and re-verify the three identities before writing back.

**Continue:** [Chapter 10 — Geometry Import/Export & Mesh Rebuilding](../C10-Geometry-IO/C10-Geometry-IO.md) ·
[Chapter 9 hub](C9-Meshes-FVF.md)
