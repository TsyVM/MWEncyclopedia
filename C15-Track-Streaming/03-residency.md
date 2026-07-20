# C15.3 — Section Residency & the Stream File

> **The one-sentence version:** the streamer keeps only the sections near you resident — it reads each needed
> section from `STREAML2RA.BUN` at the table's offset, decompresses it if the disk size is smaller than the
> memory size, and frees sections you've left — so a 532 MB world runs in a fraction of that memory.

[← C15.2 — The streaming section table](02-section-table.md) · [Chapter 15 hub](C15-Track-Streaming.md) ·
[Next: C15.4 — The world grid →](04-world-grid.md)

---

## The residency loop

Every frame, the streamer maintains a **working set** of resident sections around the player:

```
1. From the player's position, compute the set of sections that should be resident
   (a neighbourhood on the world grid — C15.4).
2. For each newly-needed section: read its payload from STREAML2RA.BUN at
   table.offset, of table.size_disk bytes; decompress to size_mem if compressed (C15.2).
3. For each no-longer-needed section: free its memory.
4. Draw the resident sections the visibility data says are visible (C15.5).
```

The section table ([C15.2](02-section-table.md)) is the map for step 2: it converts a section id into a file
offset and a byte count. The grid ([C15.4](04-world-grid.md)) drives step 1. Together they keep the city
present without ever holding all 532 MB at once.

## Why a separate 532 MB file

Splitting the payloads into `STREAML2RA.BUN` (rather than inlining them in the master file) is what makes
streaming practical:

- **Sequential, offset-addressed reads.** The stream file is one big blob addressed by offset, ideal for the
  streamer's "seek here, read this many bytes" access — friendly to the disc/HDD the game targeted.
- **The index stays small and resident.** The 1.4 MB master file ([C15.1](01-master-layout.md)) can stay in
  memory always; only the heavy payloads are streamed.
- **Compression per section.** Each section can be JDLZ-compressed independently
  ([C15.2](02-section-table.md)), decompressed only when loaded — trading a little CPU for a lot of disk and
  memory.

## Load radius vs view distance

The trick to *seamless* streaming is loading ahead of sight: the residency neighbourhood
([C15.4](04-world-grid.md)) is larger than the view distance, so a section is in memory before it can be seen.
As you drive toward it, it is already resident; as you leave, it stays briefly (hysteresis) so a U-turn
doesn't thrash. Get this radius right and streaming is invisible; too small and the world "pops in" at the
horizon.

> 🟡 *Reasoned:* the residency-loop behaviour (working set, load-ahead, hysteresis) describes how a streaming
> engine of MW's design uses the verified table; the ✅ verified facts are the table structure, the separate
> stream file, and the offset/size fields that make offset-addressed reads possible.

## Reading a section yourself

With the table, you can extract any section from the stream file exactly as the engine does:

```python
def read_section(stream_buf, entry):
    raw = stream_buf[entry["offset"] : entry["offset"] + entry["size_disk"]]
    if entry["size_disk"] < entry["size_mem"]:
        return jdlz_decompress(raw)            # compressed on disk (Chapter 3)
    return raw                                  # stored as-is

# each section's decompressed payload is itself EAGL chunks:
# SolidLists (C8), TPKs (C5), scenery (C16) — the section's world content.
```

A decompressed section is not opaque — it is more **EAGL chunks** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)):
the geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), textures
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), and scenery placements
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) that fill that patch of world. Streaming delivers the
bytes; the rest of the book decodes them.

## Editing implications

- **Offsets are absolute into the stream file.** Change a section's size and you must re-stamp its
  `+0x14` offset and every later section's offset ([C15.6](06-editing-track.md)) — the same downstream-shift
  problem as the SolidList directory ([C8.7](../C8-Geometry-Solids/07-editing.md)).
- **Keep the size fields honest.** If you recompress a section, update `size_disk`; if its memory footprint
  changes, update `size_mem`, or the streamer reads or reserves the wrong amount.
- **Don't break residency assumptions.** Wildly enlarging sections can blow the memory budget the load radius
  assumes.

---

### Key takeaways

- The streamer keeps a **working set** of nearby sections: compute needed set → read from `STREAML2RA.BUN` at
  the table offset → decompress if needed → free the rest.
- A separate offset-addressed stream file keeps the index small/resident and payloads compressible.
- Seamlessness comes from a load radius larger than view distance, with hysteresis to avoid thrashing.
- Extract a section by reading `size_disk` bytes at its offset and inflating if `size_disk < size_mem`; the
  payload is more EAGL chunks.
- Editing sizes forces offset re-stamping across later sections and honest size fields.

**Continue:** [C15.4 — The world grid](04-world-grid.md) · [Chapter 15 hub](C15-Track-Streaming.md)
