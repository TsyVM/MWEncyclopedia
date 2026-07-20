# C10.1 — The Coordinate Boundary: Z-up ↔ Y-up

> **The one-sentence version:** Most Wanted is Z-up and every mainstream 3D tool is Y-up, so export applies a
> single −90° rotation about X — engine `(x, y, z)` → tool `(x, z, −y)` — and import applies its exact
> inverse; do it once, at the boundary, and never inside the engine's own structures.

[← Chapter 10 hub](C10-Geometry-IO.md) · [Next: C10.2 — Exporting to OBJ →](02-obj-export.md)

---

## The mismatch

The engine's world is **Z-up**: Z is height, and a car's bounding box confirmed it — length in X, symmetric
width in Y, height in Z ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)). OBJ, glTF, Blender, Maya, and
3ds Max (in its export conventions) are **Y-up**: Y is height. Move geometry between the two without
accounting for this and every model lies on its side, rotated 90° about X.

## The conversion

The standard, reversible mapping is a −90° rotation about the X axis:

```
export  (engine Z-up → tool Y-up):   (x, y, z) → (x,  z, −y)
import  (tool Y-up → engine Z-up):    (x, y, z) → (x, −z,  y)
```

Applied to both **positions and normals** (normals are directions, so they rotate the same way but need no
translation). Verify the pair composes to identity — export then import must return the original coordinates
exactly — which is your guard against a sign slip:

```python
def z_up_to_y_up(p):  x, y, z = p; return (x,  z, -y)
def y_up_to_z_up(p):  x, y, z = p; return (x, -z,  y)
assert all(y_up_to_z_up(z_up_to_y_up(v)) == v for v in sample_vertices)
```

If you prefer to think in matrices, export multiplies by `Rx(−90°)` and import by `Rx(+90°)`; the tuple form
above is that rotation written out, and it is cheaper and less error-prone for a bulk vertex pass.

## Do it once, at the boundary

The cardinal rule is **convert only as data leaves for a tool and as it returns** — keep every in-engine
structure (vertices, normals, bounding boxes, transforms) in native Z-up at all times. The reasons:

- **Transforms and boxes stay coherent.** The object's placement matrix ([C8.5](../C8-Geometry-Solids/05-transform.md))
  and AABB are Z-up; if you convert vertices internally but not those, they disagree and culling/placement
  break.
- **One conversion is auditable.** A single boundary function is easy to test (the round-trip assertion
  above); conversions sprinkled through the code are impossible to reason about and double-apply.
- **Re-import is exact.** Because import is the precise inverse of export, a straight round-trip with no edits
  reproduces the original bytes' geometry.

## Winding and handedness

The −90° X rotation is a proper rotation (determinant +1), so it does **not** flip triangle winding or
handedness — a front-facing triangle stays front-facing. You therefore keep the original index winding on
export and import ([C9.4](../C9-Meshes-FVF/04-index-buffer.md)). If a tool in your pipeline additionally
mirrors an axis (some importers flip Z), that *does* invert winding, and you must flip triangle index order to
compensate. Test on a known solid: if faces render inside-out after a round-trip, a mirror crept in and you
reverse each triangle's indices.

## UV V-flip

Coordinates are not the only convention that differs. Texture V often runs top-down in one ecosystem and
bottom-up in another; OBJ/glTF and MW may disagree, showing as vertically flipped textures. If a round-tripped
model is correct in shape but its texture is upside down, flip `v → 1 − v` at the boundary
([C9.2](../C9-Meshes-FVF/02-vertex-decoded.md)). Like the coordinate rotation, apply it once and invert it on
return.

---

### Key takeaways

- MW is Z-up, tools are Y-up; export rotates `(x, y, z) → (x, z, −y)`, import inverts `(x, y, z) → (x, −z, y)`.
- Apply to positions **and** normals; assert the round-trip composes to identity.
- Convert **only at the boundary**; keep vertices, normals, boxes, and transforms native Z-up internally.
- The rotation preserves winding/handedness; only an added axis-mirror flips faces (then reverse indices).
- Watch UV V-orientation; flip `v → 1 − v` once at the boundary if textures come back upside down.

**Continue:** [C10.2 — Exporting to OBJ](02-obj-export.md) · [Chapter 10 hub](C10-Geometry-IO.md)
