# C8.4 — Bounding Boxes & the Z-up World

> **The one-sentence version:** the object header's two float-triples are an axis-aligned bounding box, and
> reading the worked car's box — 3.47 m long in X, symmetric ±0.80 m in Y, 0.53–1.22 m in Z — only makes
> sense if Z is up, which confirms Most Wanted's native **Z-up** coordinate system.

[← C8.3 — Object names & the asset hash](03-object-hash.md) · [Chapter 8 hub](C8-Geometry-Solids.md) ·
[Next: C8.5 — The placement transform →](05-transform.md)

---

## The box

At `+0x20` and `+0x30` the header stores the object's axis-aligned bounding box (AABB) as **min (x, y, z)**
and **max (x, y, z)**, each a triple of 32-bit floats followed by a padding float. For `COBALTSS_BASE_A`:

```
min = (−2.20, −0.798, 0.530)
max = (+1.27, +0.798, 1.222)
extent = (3.47, 1.595, 0.692)   # max − min
```

Those extents are the whole argument for the coordinate system. A compact car is roughly 3.5 m long, 1.6 m
wide, and (for a body panel above the chassis pivot) well under a metre tall. Map that onto the axes:

- **X = 3.47 m** → length (front-to-back).
- **Y = 1.595 m**, and *symmetric* (−0.798 to +0.798) → width (left-right, centered on the car's centerline).
- **Z = 0.692 m**, sitting *above* the origin (0.53 to 1.22) → height above the pivot.

Only a **Z-up** reading makes the numbers a car. If Y were up, the car would be 1.6 m tall and 0.7 m wide —
absurd. The symmetric Y is the clincher: a car is mirror-symmetric across its lengthwise centerline, so the
width axis straddles zero while length and height do not.

> ✅ *Verified:* the box extents (3.47 × 1.60 × 0.69 m) and the symmetric Y match a real compact car only
> under a Z-up interpretation — consistent with the engine-wide Z-up convention
> ([C1.6](../C1-EAGL-Container-Model/06-matrices-and-coordinates.md)).

## Why Z-up matters everywhere

This is not a car-only quirk; it is the world's convention, and it governs every geometry operation:

- **Height is Z.** Ground planes are constant-Z; gravity acts along −Z; a jump ramp raises Z.
- **Export must convert.** Most DCC tools (Blender, Maya, glTF) are Y-up. Exporting geometry for external
  editing means swapping axes (a Z-up → Y-up rotation), and re-importing means swapping back
  ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)). Skipping the swap lays every car on its side.
- **Do not "fix" it internally.** The book's rule is to keep the engine's Z-up coordinates untouched in all
  in-place work and convert *only* at the export boundary. Converting internally propagates axis confusion
  through the transform matrices and bounding boxes.

## Using boxes to identify and cull

The AABB is a cheap, powerful handle even before you decode a triangle:

- **Identify parts.** A thin box at one X extreme is a bumper or light; a large central box is the main body;
  a small box high in Z is a roof or antenna. Combined with per-group boxes
  ([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)) and usage names
  ([C7.4](../C7-Materials-TexAnim/04-usage-names.md)), you can map a model's structure in seconds.
- **Cull.** The engine tests the box against the view frustum to skip objects that cannot be on screen; the
  per-object box here is that test's input. When you edit geometry, the box must still enclose the object or
  it may be wrongly culled ([C8.7](07-editing.md)).
- **Sanity-check a parse.** A valid box has `min ≤ max` on every axis and modest magnitudes; wild values mean
  a misaligned header.

## Computing a box after an edit

If you move vertices ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md)), recompute the box from the new
vertex positions — in the engine's Z-up space, before any export conversion:

```python
def aabb(verts):   # verts: iterable of (x, y, z) in Z-up engine space
    xs, ys, zs = zip(*verts)
    return (min(xs), min(ys), min(zs)), (max(xs), max(ys), max(zs))
```

Write the result back to `+0x20`/`+0x30` (keeping the padding floats). A stale box is a subtle bug: the model
renders correctly when visible but pops in and out at screen edges because the culler is testing the old
extent.

---

### Key takeaways

- The header box is min/max float-triples at `+0x20`/`+0x30` (each with a padding float).
- The worked car's extents (3.47 × 1.60 × 0.69 m) with symmetric Y prove the world is **Z-up** — Z is height.
- Z-up governs everything: height is Z, and export/import must swap to/from Y-up tools; never convert
  internally.
- Boxes identify parts and drive frustum culling; keep them enclosing the geometry after edits.
- Recompute the AABB in Z-up space from new vertices and write it back with its padding.

**Continue:** [C8.5 — The placement transform](05-transform.md) · [Chapter 8 hub](C8-Geometry-Solids.md)
