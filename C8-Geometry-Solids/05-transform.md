# C8.5 — The Placement Transform

> **The one-sentence version:** each object header carries a 4×4 matrix at `+0x40` that places the solid in
> its parent's space — identity for a car's base body (it lives at the origin), and a real rotation/translation
> for placed sub-parts and world objects — stored row-major in the engine's Z-up space.

[← C8.4 — Bounding boxes & the Z-up world](04-bounding-boxes.md) · [Chapter 8 hub](C8-Geometry-Solids.md) ·
[Next: C8.6 — Finding an object: binary search →](06-lookup.md)

---

## The matrix

At `+0x40` the header stores sixteen floats — a 4×4 transformation matrix. For `COBALTSS_BASE_A` it is the
identity:

```
1 0 0 0
0 1 0 0
0 0 1 0
0 0 0 1
```

Identity is exactly right for a car's **base** object: the body sits at the model origin with no rotation and
no offset, and its vertices are already in the car's local space. The matrix earns its keep for objects that
are *placed* — a wheel positioned at each corner, a prop dropped into the world, a sub-part offset from its
parent. For those, the matrix's rotation block and translation column are non-trivial.

## Layout and conventions

The sixteen floats are a standard homogeneous transform: a 3×3 rotation/scale block plus a translation, in
the fourth row or column depending on convention. In MW's row-major, Z-up storage the practical reading is:

```
[ r00 r01 r02 0 ]     rows 0–2, cols 0–2 = rotation/scale (3×3)
[ r10 r11 r12 0 ]
[ r20 r21 r22 0 ]
[ tx  ty  tz  1 ]     row 3, cols 0–2 = translation (tx, ty, tz)
```

The bottom row holding the translation (with a 1 in the corner) is the row-vector convention: a point `p` is
transformed as `p' = p · M`. The `0/0/0/1` last column marks it as an affine (non-projective) transform. For
the identity case every off-diagonal is zero and the diagonal is 1, so both readings coincide — which is why
a base object is the safe place to *confirm* the field is a matrix, and a placed object is where you learn the
convention for that data set.

> ✅ *Verified:* the 16 floats at `+0x40` are identity for the car base (`COBALTSS_BASE_A`), consistent with a
> 4×4 placement matrix; the base-at-origin reading matches the bounding box centered sensibly on the car.
> 🟡 *Reasoned:* the exact row-vs-column translation convention is the standard MW row-major Z-up form; verify
> against a known *placed* object before applying a specific multiplication order to edited data.

## What the transform is for

- **Instancing.** The same solid can be placed multiple times with different matrices — four wheels from one
  wheel mesh, many identical props across the world. The mesh is stored once; the matrix positions each
  instance. (World-scale instancing has its own placement structures — [Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md) —
  but the per-object matrix is the same idea at the solid level.)
- **Hierarchy.** A sub-part's matrix is relative to its parent, so composing matrices down a hierarchy gives
  world placement. A door is positioned relative to the body; the body relative to the car; the car relative
  to the world.
- **Pivot.** The matrix also defines the object's pivot/origin for rotation — important for animated or
  articulated parts.

## Working with the transform

- **Leave the base at identity.** For a car body that lives at the origin, do not invent a transform — the
  vertices are already in local space and the identity is correct.
- **Keep vertices and matrix consistent.** You can express a placement either by baking it into vertex
  positions (identity matrix) or by leaving vertices local and using the matrix. Pick one; mixing them
  double-applies the transform.
- **Convert at export only.** The matrix is in Z-up space. When exporting to a Y-up tool
  ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)), convert the matrix along with the vertices, and
  convert back on import — never rewrite the stored matrix into Y-up.
- **Recompute nothing you don't have to.** If an edit doesn't move or reorient the object, leave the matrix
  untouched; it is not derived from the vertices and won't drift.

## Reading it

```python
def parse_transform(h):                 # h = object header payload
    m = struct.unpack_from("<16f", h, 0x40)
    rot = (m[0:3], m[4:7], m[8:11])     # 3×3 rotation/scale
    trans = (m[12], m[13], m[14])       # translation (row-vector convention)
    return rot, trans
```

---

### Key takeaways

- The object header holds a 4×4 placement matrix at `+0x40` (sixteen floats), in Z-up, row-major space.
- It is **identity** for a car base (vertices already local, object at origin) and non-trivial for placed
  sub-parts, wheels, and world props.
- Layout: 3×3 rotation/scale block + a translation in the bottom row, `0/0/0/1` marking an affine transform.
- The transform enables instancing and hierarchy; compose down a parent chain for world placement.
- Keep vertices and matrix consistent (don't double-apply), convert only at the export boundary, and don't
  disturb an identity base.

**Continue:** [C8.6 — Finding an object: binary search](06-lookup.md) · [Chapter 8 hub](C8-Geometry-Solids.md)
