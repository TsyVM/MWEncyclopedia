# C15.1 — The Master Track File Layout

> **The one-sentence version:** `L2RA.BUN` is the world's index — a chunk file whose `0x00034110` section
> table (720 × 92) points into the stream file, alongside per-section position, visibility, and world data
> chunks that the streamer and renderer consult.

[← Chapter 15 hub](C15-Track-Streaming.md) · [Next: C15.2 — The streaming section table →](02-section-table.md)

---

## The chunk map

Parsed from the real `TRACKS/L2RA.BUN` (1 413 824 bytes), the master file is an EAGL chunk container
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) whose top-level chunks divide into a few
functional groups:

```
0x00034191   world header / info        (96 bytes)
0x80034147   world data container       (74 636)
0x00034146   …                          (15 748)
0x00034250   section position/grid data (10 028 = 109 × 92)
0x8003B600   …                          (25 600)
0x00034110   ★ STREAMING SECTION TABLE  (66 240 = 720 × 92)
0x00034108   section data               (134 040)
0x00034109   section data               (60 552)
0x0003B800   CARP road network (WorldMapData)  (551 432)   → Chapter 18
0x8003B810   EventSequencePack                 (125 984)
0x8003B900   EventSequence / world-event data  (173 968)
0x00E34010   per-section records        (×133)
```

Three groups matter for streaming: the **section table** (`0x00034110`), the **position/grid** data
(`0x00034250`, `0x00034108/09`), and the large **CARP/event** blocks (`0x0003B800` road network,
`0x8003B810`/`0x8003B900` event-sequence data). The rest is world header and per-section detail.

## The section table is the spine

`0x00034110` is the one chunk everything else orbits: 720 entries of 92 bytes
([C15.2](02-section-table.md)), each locating a section's payload in the stream file. It is the index the
streamer reads to answer "where is section *N*, and how big is it?" The position data
([C15.4](04-world-grid.md)) tells the streamer *which* sections to load; the section table tells it *where to
get them*.

## Position and grid data

Chunk `0x00034250` (109 × 92) and the section-data chunks `0x00034108`/`0x00034109` carry the per-section
**spatial** information — where each section sits in the world and its bounds. This is what turns the player's
position into a set of section ids to stream ([C15.4](04-world-grid.md)). The recurrence of the **92-byte**
stride here and in the section table is a strong hint the two are parallel per-section tables keyed the same
way.

## The large CARP / event blocks

The largest non-table data in the master file are **not** visibility data — a correction worth stating
plainly. `0x0003B800` (551 KB) is the **CARP road network** (`WorldMapData`): its payload begins with the
`CARP` magic (stored as reversed bytes `PRAC`) and carries the `RNnd`/`RNsg`/`RNrd`/`CGrd` road-graph tags —
this is the AI/GPS graph decoded in [Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md), not
occlusion data. `0x8003B810`/`0x0003B811` are **EventSequencePack**/`EventSequenceChunk` blobs (world event
scripts), and `0x8003B900` is related event data. Visibility itself is handled by the section-level
potentially-visible set ([C15.5](05-visibility.md)) and the per-section **scenery cull tree**
([Chapter 16](../C16-Scenery-Cull/05-cull-tree.md)) — a spatial hierarchy (the `0x8003B6xx` node family in the
master file), not these `0x0003B8xx` blocks.

> ✅ *Verified:* `L2RA.BUN` is an EAGL chunk file; `0x00034110` is 66 240 bytes = 720 × 92 (the section
> table); `0x00034250` is 10 028 = 109 × 92; `0x0003B800` carries the `CARP` magic and road-graph tags
> (Chapter 18); `0x8003B810`/`0x0003B811` are event-sequence blobs.
> 🟡 *Reasoned:* the exact split of duties among the `0x8003B6xx` node family (scenery cull vs section PVS) is
> identified by structure; the CARP identity of `0x0003B800` and the section-table dimensions are verified.

## Reading the master file

```python
def open_track(path):
    buf = open(path, "rb").read()
    chunks = walk_eagl(buf)                       # Chapter 1
    section_table = chunks[0x00034110]            # 720 × 92
    position_data = chunks.get(0x00034250)        # 109 × 92
    road_network  = chunks.get(0x0003B800)        # CARP road graph (Chapter 18)
    return section_table, position_data, road_network
```

From here, decode the section table ([C15.2](02-section-table.md)) to locate stream payloads and the position
data ([C15.4](04-world-grid.md)) to know which sections a location needs.

---

### Key takeaways

- `L2RA.BUN` is an EAGL chunk file: world header, section table, position/grid data, CARP road network, and
  event-sequence blocks.
- The spine is `0x00034110` — the **720 × 92** streaming section table.
- Position data (`0x00034250` = 109 × 92, `0x00034108/09`) gives per-section spatial bounds; the 92-byte
  stride recurs.
- `0x0003B800` is the **CARP road network** (Chapter 18), not occlusion data; `0x8003B810`/`0x0003B811` are
  event-sequence blobs. Visibility uses the section PVS and the scenery cull tree (Chapter 16).
- Streaming (residency) and visibility (drawing) are separate systems that together run the open world.

**Continue:** [C15.2 — The streaming section table](02-section-table.md) · [Chapter 15 hub](C15-Track-Streaming.md)
