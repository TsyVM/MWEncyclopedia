# C1.6 — Matrices & the Z-up Coordinate System

> **The one-sentence version:** transforms are 16 floats you read straight into memory with no
> transpose, translation lives in floats 12–14, and the world is Z-up — break any of those three
> assumptions and your geometry ends up rotated, mirrored, or in orbit.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.5 — Endianness islands](05-endianness-islands.md) ·
[Next: C1.7 — Non-chunk containers →](07-non-chunk-containers.md)

---

## What it is

A transform is a 4×4 matrix of 32-bit floats, stored as 16 sequential floats. The layout is **D3D
row-major with the translation in row 3** (floats 12, 13, 14; float 15 is the homogeneous `w`, almost
always 1.0):

```
index:   0  1  2  3      row 0: right   (basis X)
         4  5  6  7      row 1: up/fwd  (basis Y)
         8  9 10 11      row 2: fwd/up  (basis Z)
        12 13 14 15      row 3: translation (x, y, z, w=1)
```

```c
float m[16];
for (int i = 0; i < 16; i++) m[i] = read_float();   // ready to use as-is
// translation = (m[12], m[13], m[14])
```

The crucial property: that exact memory order is **identical** to the column-major / column-vector
convention many math libraries (GLM, classic OpenGL) use. So a straight 16-float read produces a
correct matrix in those libraries with **no transpose**. The upper-left 3×3 is rotation/scale; row 3 is
position.

## How to read it without breaking it

- **Don't transpose.** The storage order already matches column-major math libs. Transposing "to be
  safe" is the single most common way to wreck a transform — it swaps the rotation basis with the
  translation row and turns an axis-aligned object into a sheared, rotated mess.
- **Translation is floats 12–14.** If you find a translation in floats 3/7/11 instead, you (or your
  library) have applied the *other* convention — stop and recheck rather than papering over it.
- **`w` is float 15.** It being ≈ 1.0 is a good "I'm reading the matrix aligned correctly" sanity check,
  much like the sample-rate test for [endianness](05-endianness-islands.md). If float 15 is not near
  1.0, you are almost certainly off by a few bytes (an unstripped `0x11` pad, or a mis-sized preceding
  field).
- **The 3×3 basis should be roughly orthonormal for a rigid transform.** Its three rows should each have
  length ≈ 1 and be mutually perpendicular. If they don't, either there is genuine non-uniform scale, or
  you are misreading the matrix — check the `w` canary first.

> ✅ *Verified:* the no-transpose, translation-in-row-3 layout is confirmed against the world bundles;
> reading transforms straight places props and bones where the game places them.

## The world is Z-up — and why that matters everywhere

The game's world coordinate system is **Z-up** (verified against the world bundles): the vertical axis
is Z, not Y. The engine stores *all* data in native game coordinates and performs **no axis conversion
on load or save**. That's a deliberate invariant: every parser reads raw game space, every writer writes
raw game space, and nothing in between secretly rotates the world.

Concretely, that means:

- A prop's height above the road is its **Z** coordinate.
- A car facing down a straight has its heading in the **XY** plane.
- The bounding boxes in geometry and scenery are Z-up AABBs; their `min.z`/`max.z` are floor/ceiling.

This is also a project-wide constraint, not just a file-format note: consistent Z-up end to end is the
only way the transforms from one parser stay compatible with the transforms from another. If even one
parser silently swapped Y and Z, every transform that flowed through it would be wrong relative to every
transform that didn't, and the error would be nearly impossible to localise because each parser would
look internally consistent.

## Bending it — the one legitimate place to convert, and the traps everywhere else

**The right way:** axis conversion belongs **only** in an import/export bridge to a Y-up tool (Blender,
Maya, most glTF pipelines). When you export geometry to a Y-up format, do the Y↔Z swap *in the
exporter*; when you import back, undo it *in the importer*. The file parser and the engine never see
anything but Z-up. This keeps the conversion in exactly one place where it's visible and testable — and
[Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md) builds exactly that bridge.

**The wrong way — and the symptoms:**

- **Transposing the matrix:** the object appears rotated by an unintuitive amount, often with skew if
  there was any non-uniform scale. The classic "why is my car lying on its side and stretched?" bug.
- **Converting axes in the parser instead of the bridge:** *everything* the parser touches is now Y-up
  while everything else is Z-up. Props sink into the road, bones detach, collision stops matching
  visuals — and because it's consistent within the parser, it looks like a deep engine problem rather
  than a one-line axis swap in the wrong file.
- **Treating the translation row as a direction:** if you normalise or rotate the translation row as if
  it were part of the 3×3 basis, the object teleports — translation magnitudes are world distances, not
  unit vectors, so a stray normalisation sends it to (almost) the origin or to infinity.

The discipline is simple and pays off across the entire encyclopedia: **read straight, keep Z-up,
convert only at the Y-up boundary.** Every geometry, world, camera, and cutscene chapter relies on it.

---

**Continue:** [C1.7 — The non-chunk container models](07-non-chunk-containers.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
