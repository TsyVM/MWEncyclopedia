# C10.3 — Exporting to glTF

> **The one-sentence version:** glTF is the higher-fidelity target — it carries positions, normals, UVs, and
> vertex **colors** as typed accessors, one primitive per shading group, and a scene node with the object's
> transform — so a solid round-trips with far less lost than OBJ, at the cost of a more structured writer.

[← C10.2 — Exporting to OBJ](02-obj-export.md) · [Chapter 10 hub](C10-Geometry-IO.md) ·
[Next: C10.4 — What OBJ/glTF can't carry →](04-lossy-boundaries.md)

---

## Why glTF over OBJ

glTF (the GL Transmission Format) is a modern, typed mesh container, and it happens to fit MW's mesh model
well:

- **Per-vertex attributes** including `COLOR_0`, so vertex colors ([C9.2](../C9-Meshes-FVF/02-vertex-decoded.md))
  survive — the biggest thing OBJ drops.
- **Multiple primitives per mesh**, each with its own material — a natural home for shading groups
  ([C9.5](../C9-Meshes-FVF/05-group-ranges.md)), one primitive per group.
- **A scene graph with node transforms**, so the object's placement matrix
  ([C8.5](../C8-Geometry-Solids/05-transform.md)) can be represented instead of baked away.
- **Typed accessors** (`FLOAT` vec3 positions/normals, `FLOAT` vec2 UVs, `UNSIGNED_SHORT` indices) that match
  MW's data almost exactly — including the `u16` index type ([C9.4](../C9-Meshes-FVF/04-index-buffer.md)).

## The structure

A solid becomes one glTF mesh with one primitive per group, all sharing vertex accessors:

```
glTF
├── scene → node (TRS from the object transform, Y-up)
│            └── mesh
├── mesh
│   └── primitives[] (one per shading group)
│        ├── attributes: POSITION, NORMAL, TEXCOORD_0, COLOR_0
│        ├── indices: accessor of UNSIGNED_SHORT (this group's triangles)
│        └── material: index into materials[]
├── accessors: POSITION (vec3), NORMAL (vec3), TEXCOORD_0 (vec2), COLOR_0 (vec4), + one indices accessor/group
├── bufferViews / buffer: the packed vertex + index bytes
└── materials[]: one per group, base-color texture = the group's extracted texture
```

Positions and normals are written **Y-up** ([C10.1](01-coordinate-boundary.md)); the node's transform, if you
carry it, is likewise converted. Indices are per-primitive: split the solid's index buffer by group range and
give each primitive its own indices accessor.

## Writing it

Rather than hand-roll JSON + binary, build the structure with a glTF library (for example `pygltflib`) and let
it manage accessors and buffer views:

```python
def export_gltf(solid, path):
    mesh = assemble_solid(solid)                                  # C9.6
    pos   = [z_up_to_y_up(v["pos"])    for v in mesh["vertices"]] # C10.1
    nrm   = [z_up_to_y_up(v["normal"]) for v in mesh["vertices"]]
    uv    = [(u, 1.0 - vv) for (u, vv) in (v["uv"] for v in mesh["vertices"])]
    col   = [unpack_color_to_float4(v["color"]) for v in mesh["vertices"]]  # C9.2
    prims = []
    for sm in mesh["submeshes"]:
        idx = [i for tri in sm["triangles"] for i in tri]        # flatten, u16
        prims.append(make_primitive(pos, nrm, uv, col, idx,
                                    material=material_for(sm)))   # C7.4 name + texture
        # keep sm["texture_key"] in the sidecar (C10.4)
    write_gltf(path, prims, node_transform=convert_transform(solid))
```

## What glTF still can't hold natively

glTF preserves far more than OBJ, but two MW-specific things still have no standard slot and go in the
side-channel ([C10.4](04-lossy-boundaries.md)):

- **The texture asset key.** glTF materials reference an image *file*, not MW's 32-bit key
  ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)); keep a material-name → key map so re-import can
  restore the binding.
- **The exact vertex encoding.** glTF stores normals as full floats; MW's stride-60 tangent frames and any
  packed encodings ([C9.3](../C9-Meshes-FVF/03-fvf-strides.md)) must be re-derived or carried out-of-band.

Even so, glTF's native color and multi-material support means the side-channel is smaller than with OBJ, which
is the main reason to prefer it for anything beyond a quick reshape.

## Choosing OBJ vs glTF

- **OBJ** — quick edits, universal tool support, human-readable; accept color/UV/graph loss.
- **glTF** — vertex colors, multiple materials, scene transform, `u16` indices matching MW; more setup, less
  loss. Prefer it for vertex-colored, normal-mapped, or multi-part geometry.

Both still require the coordinate conversion, the side-channel, and the size-tree fixups on the way back
([C10.5](05-reimport-rebuild.md)–[C10.6](06-sizetree-verify.md)); glTF simply loses less in the middle.

---

### Key takeaways

- glTF carries positions, normals, UVs, **vertex colors**, per-group primitives, and a node transform — a
  close fit to MW's mesh model.
- One glTF mesh with one primitive per shading group; `UNSIGNED_SHORT` indices match MW's `u16`.
- Write positions/normals Y-up; split indices per group; one material per group.
- Still keep the **asset key** and any packed normal/tangent encoding in the side-channel.
- Prefer glTF over OBJ when color, multiple materials, or structure matter.

**Continue:** [C10.4 — What OBJ/glTF can't carry](04-lossy-boundaries.md) · [Chapter 10 hub](C10-Geometry-IO.md)
