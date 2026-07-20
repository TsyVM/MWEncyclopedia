# C10.2 — Exporting to OBJ

> **The one-sentence version:** map a solid onto OBJ by writing `v`/`vt`/`vn` from the decoded vertices
> (converted to Y-up) and one `usemtl` face block per shading group, with a companion `.mtl` naming each
> material — a lossy but universal export good enough to see and reshape geometry anywhere.

[← C10.1 — The coordinate boundary](01-coordinate-boundary.md) · [Chapter 10 hub](C10-Geometry-IO.md) ·
[Next: C10.3 — Exporting to glTF →](03-gltf-export.md)

---

## The mapping

OBJ's primitives line up with a decoded solid almost one-to-one:

| MW mesh element | OBJ element |
|---|---|
| vertex position ([C9.2](../C9-Meshes-FVF/02-vertex-decoded.md)) | `v x y z` (Y-up) |
| vertex UV | `vt u v` |
| vertex normal | `vn x y z` (Y-up) |
| shading group ([C9.5](../C9-Meshes-FVF/05-group-ranges.md)) | `usemtl <name>` + `f` block |
| group's texture/usage name ([C7.4](../C7-Materials-TexAnim/04-usage-names.md)) | `.mtl` material entry |

The two things OBJ does not have — vertex **color** and the texture **asset key** — go in the side-channel
([C10.4](04-lossy-boundaries.md)); everything else is a direct write.

## The writer

```python
def export_obj(solid, obj_path, mtl_path):
    mesh = assemble_solid(solid)                 # C9.6
    with open(obj_path, "w") as f:
        f.write(f"mtllib {mtl_path}\n")
        # positions, UVs, normals — converted Z-up → Y-up (C10.1)
        for v in mesh["vertices"]:
            x, y, z = z_up_to_y_up(v["pos"]);  f.write(f"v {x} {y} {z}\n")
        for v in mesh["vertices"]:
            u, vv = v["uv"];                   f.write(f"vt {u} {1.0 - vv}\n")   # V-flip
        for v in mesh["vertices"]:
            nx, ny, nz = z_up_to_y_up(v["normal"]); f.write(f"vn {nx} {ny} {nz}\n")
        # one material block per shading group
        for i, sm in enumerate(mesh["submeshes"]):
            f.write(f"usemtl {sm['usage_name'] or f'mat_{i}'}\n")
            for (a, b, c) in sm["triangles"]:
                # OBJ is 1-indexed; v/vt/vn share the index here
                f.write(f"f {a+1}/{a+1}/{a+1} {b+1}/{b+1}/{b+1} {c+1}/{c+1}/{c+1}\n")
    write_mtl(mtl_path, mesh["submeshes"])
```

Three details make or break an OBJ export:

- **1-indexing.** OBJ indices start at 1, not 0 — add one to every index, or the whole model is off by a
  vertex.
- **Y-up conversion.** Positions and normals go through `z_up_to_y_up` ([C10.1](01-coordinate-boundary.md));
  UVs get the `1 − v` flip if your target expects it.
- **Winding.** Emit the triangle's indices in their original order to preserve facing
  ([C9.4](../C9-Meshes-FVF/04-index-buffer.md)).

## The companion `.mtl`

Each shading group becomes a material entry. Name it by the usage name so the classification survives the
trip ([C7.4](../C7-Materials-TexAnim/04-usage-names.md)), and point `map_Kd` at the texture you extracted for
that group's key ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[6](../C6-Texture-Codecs/C6-Texture-Codecs.md)):

```python
def write_mtl(path, submeshes):
    with open(path, "w") as f:
        for i, sm in enumerate(submeshes):
            name = sm["usage_name"] or f"mat_{i}"
            f.write(f"newmtl {name}\n")
            f.write("Kd 1 1 1\n")                       # base color
            if sm.get("texture_png"):
                f.write(f"map_Kd {sm['texture_png']}\n")  # the extracted texture
            f.write("\n")
```

Extract each referenced texture to PNG/DDS first so the `.mtl`'s `map_Kd` resolves; the material name carries
the usage classification, and the sidecar carries the numeric asset key for re-import.

## OBJ's honest limits

OBJ is the right choice when you want *universality and simplicity* — every tool reads it, and it is
human-readable for debugging. Accept that it drops vertex color, supports a single UV set, has no scene graph
or transform, and re-derives normals in many importers. For a car panel you want to reshape and put back, that
is usually fine, provided you keep the side-channel ([C10.4](04-lossy-boundaries.md)). When the losses matter
— vertex-colored surfaces, normal-mapped panels, multi-part hierarchies — prefer glTF ([C10.3](03-gltf-export.md)).

---

### Key takeaways

- OBJ maps directly: `v`/`vt`/`vn` from decoded vertices, one `usemtl` face block per shading group, plus a
  `.mtl`.
- Remember the three gotchas: **1-indexing**, **Y-up conversion** of positions/normals, and preserved
  **winding**.
- Name materials by usage name and point `map_Kd` at the extracted texture; keep the asset key in the
  side-channel.
- OBJ drops color, extra UVs, scene graph, and exact normals — fine for reshaping, but keep the sidecar.
- Choose glTF when those losses matter.

**Continue:** [C10.3 — Exporting to glTF](03-gltf-export.md) · [Chapter 10 hub](C10-Geometry-IO.md)
