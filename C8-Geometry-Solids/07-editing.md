# C8.7 — Editing Solids Safely

> **The one-sentence version:** a SolidList has *two* consistency systems to respect — the chunk size tree
> (every container size) and the directory (every object's stored offset and size) — so any edit that changes
> an object's byte length must fix the size tree *and* re-stamp every downstream directory offset, or lookup
> silently seeks into the wrong place.

[← C8.6 — Finding an object: binary search](06-lookup.md) · [Chapter 8 hub](C8-Geometry-Solids.md) ·
[Next: Chapter 9 — Meshes, FVF & Vertex Formats →](../C9-Meshes-FVF/C9-Meshes-FVF.md)

---

## Two bookkeeping systems, not one

Editing a texture pack meant respecting one invariant — the size tree ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)).
A SolidList adds a second: the **object directory** (`0x00134004`) stores an absolute file offset and a byte
size for every object ([C8.1](01-solidlist-container.md)). So a solid's identity and location are recorded in
*two* places that must agree with reality:

1. **The size tree** — the chunk size headers from the edited object up through `0x80134010`,
   `0x80134000`, and the containing bundle.
2. **The directory** — the `{offset, size}` row for the edited object, *and* the offsets of every object
   stored **after** it (because they all shift when a preceding object changes length).

Miss the first and the tree walk desynchronises; miss the second and binary-search lookup
([C8.6](06-lookup.md)) jumps to a stale offset and reads garbage. Both failures are silent until the game
loads the file.

## The safe edit: keep the size constant

As always, the safest edit changes nothing's length. If you modify an object *in place* without changing its
byte size — recoloring via material references ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)),
tweaking a bounding box, editing a transform, moving a vertex without adding/removing any — then:

- no chunk size changes, so the **size tree is untouched**;
- no object moves, so **every directory offset stays valid**;
- the object count is unchanged, so the **three-way count check still holds**.

This is by far the most reliable class of geometry edit, and a great deal is possible within it: all of an
object's *values* (positions, normals, UVs, colors, material links, box, transform) can change as long as the
*counts and sizes* do not.

## The repack edit: when length changes

Adding or removing vertices or triangles, or adding an object, changes byte lengths and triggers full
bookkeeping:

1. **Rebuild the changed object** and note its new size.
2. **Fix its chunk size headers** up the tree: mesh buffers → `0x80134100` → `0x80134010` → `0x80134000` →
   containing bundle.
3. **Re-stamp the directory.** Update the edited object's `{offset, size}` row, then walk every *later*
   object and add the byte delta to its stored offset. Only objects after the edit move, but recomputing the
   whole directory from the actual chunk positions is simplest and least error-prone.
4. **Keep the header counts truthful.** If you changed triangle or vertex counts, update the object header's
   `+0x14` (and the mesh descriptor) so `num_tris × 6 + 8 == indexBufferSize` still holds
   ([C8.2](02-object-header.md)).
5. **Maintain the sorted hash table** if you add/remove objects: insert/remove the name-hash *in sorted
   position*, update the count in three places, and keep the directory in the matching index space
   ([C8.1](01-solidlist-container.md)). Break the sort and binary search fails.

```python
def restamp_directory(objects):
    # objects: list in storage order, each with .bytes (already size-fixed)
    cursor = first_object_offset
    for o in objects:
        o.dir_offset = cursor          # absolute offset of this object's 0x80134010
        o.dir_size   = len(o.bytes)
        cursor      += len(o.bytes)
    # then write each (hash, dir_offset, dir_size) back into 0x00134004,
    # and ensure 0x00134003 stays sorted by hash with matching indices
```

## Bounding boxes and counts must stay honest

Two derived quantities are easy to forget and cause subtle, load-time-clean-but-wrong behaviour:

- **The bounding box** must still enclose the (possibly moved) vertices, or the culler mis-handles the object
  ([C8.4](04-bounding-boxes.md)). Recompute it in Z-up space and write it back.
- **The triangle/vertex counts** in the header and mesh descriptor must match the buffers exactly
  ([C7.1](../C7-Materials-TexAnim/01-mesh-container.md)); a count that disagrees with a buffer size is the
  classic "half the model is missing / the game crashes on load" bug.

## Verify by re-parsing

After any structural edit, re-open the file and confirm, in order:

1. the SolidList's **three-way count** still agrees (header field, `hashtable/8`, `directory/24`);
2. every **directory offset** lands on a `0x80134010` chunk;
3. the **hash table is still sorted** ascending;
4. each object's **`num_tris × 6 + 8`** equals its index-buffer size;
5. the outer **size tree** walks cleanly to the end of the bundle.

If all five hold, the SolidList is internally consistent and loadable. This checklist is the geometry
equivalent of the TPK "offsets tile the blob" verification ([C5.5](../C5-Textures-TPK/05-extract-replace.md)) —
cheap to run, and it catches every common editing mistake before the game sees the file.

---

### Key takeaways

- A SolidList has **two** consistency systems: the chunk size tree and the object directory (offsets + sizes).
- Same-size in-place edits (values, not counts) touch neither system and are the safe path.
- Length-changing edits require size-tree fixups **and** re-stamping every downstream directory offset, plus
  honest header counts.
- Adding/removing objects means maintaining the **sorted** hash table and the three-way count in lockstep.
- Recompute bounding boxes and keep `num_tris × 6 + 8 == indexBufferSize`; verify by the five-point re-parse.

**Continue:** [Chapter 9 — Meshes, FVF & Vertex Formats](../C9-Meshes-FVF/C9-Meshes-FVF.md) ·
[Chapter 8 hub](C8-Geometry-Solids.md)
