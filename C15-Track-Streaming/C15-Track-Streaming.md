# Chapter 15 — The Master Track File & Streaming Sections

> **Goal of this chapter:** open the master track file, find the streaming section table, and understand how
> Most Wanted's open world is divided into hundreds of sections that stream in and out of memory from a giant
> companion file — the backbone of the seamless city.

Most Wanted's world is far too large to hold in memory at once, so it is **streamed**: divided into sections,
each loaded when you approach and freed when you leave. The **master track file** (`TRACKS/L2RA.BUN`) is the
world's index and top-level data; the **stream file** (`TRACKS/STREAML2RA.BUN`, ~532 MB) holds the section
payloads. This chapter decodes the section table that ties them together and the residency model that keeps
the city seamless.

> **Verified against retail data.** The master file `TRACKS/L2RA.BUN` (1 413 824 bytes) was parsed to its
> chunk map, and its streaming section table — chunk **`0x00034110`** — is exactly **720 entries of 92
> bytes** (66 240 bytes). Each entry carries a section id, an offset into the stream file, size fields, and a
> section hash, decoded below. The stream file `STREAML2RA.BUN` is 532 663 556 bytes.

---

## Deep-dive pages

- [C15.1 — The master track file layout](01-master-layout.md): the chunk map of `L2RA.BUN` — section table,
  position data, visibility, and world chunks.
- [C15.2 — The streaming section table](02-section-table.md): the 92-byte entry decoded — id, stream offset,
  sizes, hash.
- [C15.3 — Section residency & the stream file](03-residency.md): how sections load from `STREAML2RA.BUN` and
  are freed.
- [C15.4 — The world grid](04-world-grid.md): how sections tile the map and how position selects them.
- [C15.5 — Visibility & culling data](05-visibility.md): the barrier/visibility blocks that decide what a
  section can see.
- [C15.6 — Working with track data safely](06-editing-track.md): the size-tree and offset discipline for
  world edits.
- [C15.7 — The anatomy of a stream section](07-section-contents.md): the per-section chunk bundle — geometry,
  textures, scenery, and collision — and the alignment invariant that binds it.

---

## 15.1 Two files, one world

The world is split across two files with distinct jobs:

- **`L2RA.BUN`** (1.4 MB) — the **master track file**: the section table, per-section position and visibility
  data, and the top-level world chunks. Small enough to stay resident; it is the *index*.
- **`STREAML2RA.BUN`** (532 MB) — the **stream file**: the actual geometry, textures, and data of each
  section, addressed by offset. It is the *content*, streamed on demand.

The master file's section table points into the stream file: each entry says "section *N* lives at this offset
in `STREAML2RA.BUN`, is this big, and has this id." That indirection is what lets a 532 MB world run in a
fraction of that memory ([C15.3](03-residency.md)).

## 15.2 The section table: 720 × 92

The heart of the master file is chunk **`0x00034110`**, a flat table of **720 sections**, each a **92-byte**
entry. Decoded from real data, an entry carries:

| Offset | Field | Role |
|---|---|---|
| `+0x00` | section id | the section's identifier (e.g. `0x3058`) |
| `+0x14` | stream offset | byte offset of the section in `STREAML2RA.BUN` |
| `+0x18`/`+0x1C`/`+0x20` | sizes | payload sizes (data streams within the section) |
| `+0x24` | count/flags | a per-section count |
| `+0x34` | section hash | the section's asset hash |

720 sections × 92 bytes = 66 240 bytes exactly, matching the chunk size — the divide-to-N sanity check you
know from TPKs and SolidLists ([C15.2](02-section-table.md)).

## 15.3 Residency keeps the city seamless

The engine keeps only the sections near you resident. As you drive, it uses your position to decide which
sections should be loaded ([C15.4](04-world-grid.md)), reads those from `STREAML2RA.BUN` at the offsets the
table gives, and frees sections you've left. Done well — with a load radius larger than the view distance —
this is invisible: the world is always there before you can see it. The section table is the map the streamer
consults every frame ([C15.3](03-residency.md)).

## 15.4 The world is a grid of sections

Sections tile the map spatially, so "which sections do I need?" is answered from your world position: the
sections in a neighbourhood around you. The master file carries the per-section position/bounds data that lets
the streamer translate "I am here" into "load these section ids" ([C15.4](04-world-grid.md)), and the
visibility blocks refine what each resident section can actually see and draw ([C15.5](05-visibility.md)).

---

### Key takeaways

- The world is two files: `L2RA.BUN` (master index, 1.4 MB) and `STREAML2RA.BUN` (stream payloads, 532 MB).
- The section table is chunk **`0x00034110`**: **720 sections × 92 bytes**, each with id, stream offset,
  sizes, and hash.
- The table indexes payloads in the stream file, enabling a huge world in modest memory.
- Residency streams sections near the player in and frees distant ones, keeping the city seamless.
- Sections tile the map as a grid; position selects which to load, and visibility data refines drawing.

**Next:** [Chapter 16 — Scenery, Props & the Cull Tree](../C16-Scenery-Cull/C16-Scenery-Cull.md): what fills
the sections this streaming system delivers.
