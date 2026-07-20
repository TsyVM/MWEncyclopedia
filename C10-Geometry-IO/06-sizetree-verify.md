# C10.6 — Size-Tree Consequences & Verification

> **The one-sentence version:** a rebuilt mesh has new buffer sizes, so import ends where all MW editing does
> — propagate the deltas up the chunk tree, re-stamp the object directory, and prove the result with the
> five-point re-parse before the game ever sees it.

[← C10.5 — Re-importing & rebuilding buffers](05-reimport-rebuild.md) · [Chapter 10 hub](C10-Geometry-IO.md) ·
[Next: Chapter 11 — Attribute Vaults →](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)

---

## New buffers, new sizes

Unless your edit preserved the exact vertex and triangle counts, the rebuilt vertex and index buffers differ
in length from the originals, and that delta must ripple outward through every size and offset that describes
them. This is the geometry instance of the size-tree discipline
([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)) combined with the SolidList's second
bookkeeping system, the object directory ([C8.7](../C8-Geometry-Solids/07-editing.md)).

## The propagation chain

Fix sizes and offsets from the innermost change outward, in order:

```
0x00134B01 / 0x00134B03  (rebuilt buffers)     ← new sizes
        ↓
0x80134100  mesh container                     ← size = Σ children
        ↓
0x80134010  solid                              ← size = Σ children
        ↓
0x80134000  SolidList                          ← size = Σ children
        ↓
0x00134004  object directory                   ← re-stamp this object's size,
                                                  and every later object's OFFSET
        ↓
containing bundle chunk                        ← size += total delta
```

Two steps are the ones people miss:

- **The object directory.** The edited object's `{offset, size}` row changes, and because the object grew or
  shrank, **every object stored after it shifts** — add the byte delta to each later object's directory offset
  ([C8.1](../C8-Geometry-Solids/01-solidlist-container.md)). Skip this and binary-search lookups
  ([C8.6](../C8-Geometry-Solids/06-lookup.md)) seek into the wrong place.
- **The containing bundle.** The SolidList lives inside a bundle whose chunk header also states a size; the
  total delta must reach it, or the outer file's tree walk desynchronises
  ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)).

## Counts, boxes, and hashes stay honest

Alongside sizes, three derived quantities must be updated or the file loads but misbehaves:

- **Header counts** — `num_tris` (`+0x14`) and the mesh descriptor's index count (`+0x2C`) must match the new
  buffers ([C8.2](../C8-Geometry-Solids/02-object-header.md)).
- **Bounding boxes** — the object box ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)) and each group box
  ([C7.2](../C7-Materials-TexAnim/02-shading-groups.md)) must enclose the new vertices; recompute in Z-up.
- **Name-hash** — if you renamed the object, its hash changes; keep the header hash, the sorted hash table,
  and the directory in agreement, and re-sort the table if a hash moved
  ([C8.3](../C8-Geometry-Solids/03-object-hash.md)).

## The five-point verification

Re-open the file you just wrote and confirm, in order — the geometry counterpart of the TPK's "offsets tile
the blob" check ([C5.5](../C5-Textures-TPK/05-extract-replace.md)):

1. **Three-way count** agrees — list-header field, `hashtable / 8`, `directory / 24`
   ([C8.1](../C8-Geometry-Solids/01-solidlist-container.md)).
2. **Every directory offset** lands on a `0x80134010` chunk.
3. **Hash table sorted** ascending ([C8.6](../C8-Geometry-Solids/06-lookup.md)).
4. **`num_tris × 6 + 8 == indexBufferSize`** for the edited object, and vertex count keeps all indices in
   range ([C9.6](../C9-Meshes-FVF/06-assembling.md)).
5. **Outer size tree** walks cleanly to the end of the bundle ([C1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)).

If all five hold, the rebuilt solid is internally consistent and the bundle is loadable.

```python
def verify_solidlist(buf):
    n, fname, hashes, directory = read_solidlist_header(header_chunks(buf))
    assert len(hashes) == n == len(directory)                      # (1)
    assert all(chunk_id_at(buf, off) == 0x80134010 for _,off,_ in directory)  # (2)
    assert hashes == sorted(hashes)                                # (3)
    for solid in iter_solids(buf):                                 # (4)
        assert solid.num_tris*6 + 8 == len(solid.index_buffer)
        assert max(solid.all_indices) < solid.vertex_count
    assert walks_to_end(buf)                                       # (5)
```

## Do the safe edit when you can

Everything above is the *repack* path, needed when counts change. If your edit can keep the vertex and
triangle counts constant — reshaping without adding/removing geometry, recoloring, retexturing, moving
vertices in place — then **no size changes**, the directory is untouched, and you skip almost all of this
([C8.7](../C8-Geometry-Solids/07-editing.md)). Design edits to preserve counts whenever the result allows; it
turns a delicate multi-structure repack into a byte-for-byte-sized overwrite.

---

### Key takeaways

- A rebuilt mesh's new buffer sizes propagate: buffers → mesh container → solid → SolidList → directory →
  bundle.
- Re-stamp the edited object's directory row **and** every later object's offset; carry the delta to the
  containing bundle.
- Keep header counts, bounding boxes, and (if renamed) the sorted hash table honest.
- Verify with the five-point re-parse: three-way count, directory offsets, sorted table, tri/index identity,
  outer tree walk.
- Prefer count-preserving edits to avoid the repack entirely.

**Continue:** [Chapter 11 — Attribute Vaults: VPAK Structure](../C11-Attribute-Vaults/C11-Attribute-Vaults.md) ·
[Chapter 10 hub](C10-Geometry-IO.md)
