# C15.6 — Working with Track Data Safely

> **The one-sentence version:** editing the world means respecting three linked systems at once — the chunk
> size tree, the section table's absolute stream offsets, and the precomputed visibility — so a section
> whose size changes forces offset re-stamping across the stream file and, for big changes, a visibility
> rebuild.

[← C15.5 — Visibility & culling data](05-visibility.md) · [Chapter 15 hub](C15-Track-Streaming.md) ·
[Next: Chapter 16 — Scenery, Props & the Cull Tree →](../C16-Scenery-Cull/C16-Scenery-Cull.md)

---

## Three systems to keep consistent

World edits are the most bookkeeping-heavy in the game because three systems describe the same content:

1. **The chunk size tree** — inside the master file and inside each section's EAGL payload
   ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)).
2. **The section table** — absolute offsets and sizes into `STREAML2RA.BUN`
   ([C15.2](02-section-table.md)).
3. **The visibility/PVS data** — computed for the original geometry ([C15.5](05-visibility.md)).

An edit that changes a section's bytes touches all three: the size tree within the section, the table's offset
and size fields for that section *and every later section*, and potentially the PVS if geometry moved.

## The safe edit: change content, not size

As everywhere in this book, the safest world edit keeps sizes constant. Editing values *within* a section's
existing structure — retexturing a surface ([C5.5](../C5-Textures-TPK/05-extract-replace.md)), recoloring,
nudging a vertex without adding/removing geometry ([C8.7](../C8-Geometry-Solids/07-editing.md)) — changes no
byte counts, so:

- the section's payload size is unchanged, so **its stream offset and every later offset stay valid**;
- the section table needs no edits;
- the PVS remains valid (geometry didn't structurally move).

Design world edits to stay size-neutral whenever possible; it turns a cross-file repack into a contained
overwrite.

## The repack edit: sizes cascade across the stream file

If a section's payload changes size, the cascade is the largest in the game:

1. **Fix the section's internal size tree** (its SolidLists, TPKs, etc.).
2. **Update the section table entry** — new `size_disk`/`size_mem` ([C15.2](02-section-table.md)).
3. **Re-stamp `+0x14` stream offsets for every *later* section** — because sections are packed sequentially in
   `STREAML2RA.BUN`, growing one shifts all that follow, exactly like the SolidList directory
   ([C8.7](../C8-Geometry-Solids/07-editing.md)) but across a 532 MB file.
4. **Rewrite the stream file** with the new payload and shifted sections.
5. **Rebuild visibility** if the change was structural ([C15.5](05-visibility.md)).

```python
def restamp_stream(sections):
    cursor = first_section_offset
    for s in sections:                 # in stream order
        s["offset"]    = cursor
        s["size_disk"] = len(s["payload_on_disk"])
        cursor        += s["size_disk"]
    # write each section's payload at its new offset in STREAML2RA.BUN,
    # and write the updated 92-byte table entries back into 0x00034110
```

The scale is what makes this daunting — a single early-section size change re-stamps hundreds of later
offsets and rewrites much of a half-gigabyte file. This is why practical world modding leans hard on
size-neutral edits and section-local changes.

## Compression matters here

Because sections can be JDLZ-compressed ([C15.2](02-section-table.md)), a content edit that would be
size-neutral *uncompressed* may change the *compressed* size — and it is the compressed (`size_disk`) size
that determines packing in the stream file. So even a "same uncompressed size" edit can force an offset
cascade after recompression. When you must edit a compressed section, either recompress and re-stamp, or store
it uncompressed (larger, but stable) if the memory budget allows.

## Verify across the whole chain

After a world edit, verify in layers:

1. **Section payload** — its internal EAGL tree walks cleanly ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)).
2. **Section table** — every `+0x14` offset lands at the start of a valid section in `STREAML2RA.BUN`, and
   sizes match the payloads.
3. **Coverage** — offsets tile the stream file with no gaps/overlaps (the streaming analogue of "offsets tile
   the blob," [C5.5](../C5-Textures-TPK/05-extract-replace.md)).
4. **In game** — drive the area: the world streams in without pop-in or holes, and nothing vanishes that
   should be visible (a PVS check).

## Editing implications, summarised

- **Prefer size-neutral, section-local edits.** They avoid the cross-file cascade entirely.
- **Any size change re-stamps all later stream offsets** and updates the table — plan for it.
- **Watch compression** — compressed size drives packing, so recompression can force a cascade.
- **Rebuild visibility for structural changes**, and verify by driving the area.

---

### Key takeaways

- World edits must keep three systems consistent: the chunk size tree, the section table's absolute offsets,
  and the visibility/PVS.
- Size-neutral, section-local edits touch none of the cross-file bookkeeping — strongly prefer them.
- A section size change re-stamps `+0x14` offsets for every later section and rewrites the stream file.
- Compression complicates this: `size_disk` drives packing, so recompression can trigger the cascade.
- Verify payload trees, table offsets, stream coverage, and in-game streaming; rebuild visibility for
  structural edits.

**Continue:** [Chapter 16 — Scenery, Props & the Cull Tree](../C16-Scenery-Cull/C16-Scenery-Cull.md) ·
[Chapter 15 hub](C15-Track-Streaming.md)
