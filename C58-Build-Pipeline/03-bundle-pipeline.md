# C58.3 — The Bundle Pipeline

> **The one-sentence version:** the game's assets ship as bundles of chunks — the `BCHUNK` system built on the EAGL
> `{id, size}` chunk container — holding textures, geometry, and vault data, JDLZ/LZC-compressed, the shipping
> format the whole book has been parsing.

[← C58.2 — The EAGL engine](02-eagl-engine.md) · [Chapter 58 hub](C58-Build-Pipeline.md) ·
[Next: C58.4 — The asset pipeline →](04-asset-pipeline.md)

---

## Bundles of chunks

Most Wanted's assets don't ship as loose files — they ship as **bundles**, and a bundle is a tree of **chunks**.
The verified `BCHUNK_` system (`BCHUNK_NULL`) is EA Black Box's chunk format, built on the **EAGL chunk container**
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)):

- **A chunk** is `{u32 id, u32 size}` + payload ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md))
  — a typed, sized block. `BCHUNK_NULL` is a null/padding chunk.
- **Chunks nest** — a container chunk holds child chunks, forming a tree
  ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)).
- **A bundle** is a chunk tree in a file — e.g. the `GLOBAL`/`GLOBALB` bundle
  ([C58.1](01-shipping-exe.md)) holding globally-needed assets.

So the shipping data is a hierarchy of typed chunks — the exact structure the whole book has parsed: the TPK
texture chunks ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), the SolidList geometry
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), the vault
([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)), the audio banks
([Chapter 19](../C19-Audio-Banks/C19-Audio-Banks.md)) — all `BCHUNK` chunks in bundles.

> ✅ *Verified:* `BCHUNK_` and `BCHUNK_NULL` are present in `speed.exe` — the bundle-chunk system. It's built on
> the EAGL `{id, size}` chunk container ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)); the
> `GLOBAL`/`GLOBALB` bundle is named.

## The chunk IDs are the format map

Every asset type the book decoded has a **chunk ID** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) —
the `id` field that says what a chunk *is*:

| Chunk id | Asset | Chapter |
|---|---|---|
| `0xB3300000` | TPK texture pack | [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md) |
| `0x80134000` | SolidList geometry | [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md) |
| … | vault, audio, animation, … | [Chapters 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)+ |

So the chunk-ID space is the *format map* of the whole game — each ID a decoded format
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)). Parsing a bundle is walking its chunk tree,
dispatching each chunk to its format decoder by ID. This is the foundation the *entire first half of the book*
([Parts I–VI](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) rests on — every asset format is a chunk type,
and the bundle is the chunk container that holds them. The `BCHUNK` bundle is thus the *root* of the data side of
the game.

## Compression: JDLZ and LZC

Bundles are **compressed** to fit the 2005 disc/memory budget ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)):

- **JDLZ** (magic `JDLZ`) — the primary compression ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) —
  a fast LZ variant, decompressed on load.
- **LZC** — the `.LZC` compressed bundles (e.g. the global bundle) — another compression layer.

So a shipped bundle is *compressed chunks* — the file on disc is JDLZ/LZC-compressed, decompressed into the chunk
tree at load ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)). This is why
the streaming system ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) has a
decompression step — the bundles arrive compressed and must be expanded. Compression is the last pipeline stage
before shipping ([C58.4](04-asset-pipeline.md)): pack the chunks, compress the bundle, ship the disc. It's what let
a huge open world ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) fit on a 2005 DVD.

## Why bundles of chunks

The bundle-of-chunks format ([above](#bundles-of-chunks)) is a clean, extensible shipping design:

- **Typed and self-describing.** Each chunk's `id` says what it is and its `size` says how big
  ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — so a loader can walk a bundle, skip
  unknown chunks, and dispatch known ones. The format is *extensible* (add a chunk type without breaking old
  loaders).
- **Hierarchical.** The chunk tree ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) mirrors the
  asset structure — a car bundle holds geometry, texture, and vault chunks together.
- **Streamable and compressible.** Bundles are units of streaming
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) and compression
  ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) — load and decompress a bundle to get all its
  chunks.

So the bundle-of-chunks format is the *shipping container* of the whole game — typed, hierarchical, streamable,
compressible. It's the format the entire runtime consumes and the entire book decodes. Understanding it
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) is understanding *how the game's data is
structured* — the single most foundational fact, from which every asset format follows. The `BCHUNK` bundle is
where the *pipeline* ([C58.4](04-asset-pipeline.md)) delivers, and where the *runtime* begins.

## RE implications

- **Assets ship as bundles of `BCHUNK` chunks** — the EAGL `{id, size}` container
  ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)).
- **Chunk IDs are the format map** — each ID a decoded asset type
  ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)+).
- **Bundles are JDLZ/LZC-compressed** ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) — decompressed on
  load.
- **Typed, hierarchical, streamable, compressible** — the shipping container the whole runtime consumes.

---

### Key takeaways

- Assets ship as **bundles of `BCHUNK` chunks** — EA Black Box's format on the EAGL `{id, size}` chunk container
  ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — e.g. the `GLOBAL`/`GLOBALB` bundle.
- The **chunk IDs are the format map** of the game — `0xB3300000` (TPK textures), `0x80134000` (SolidList
  geometry), etc. — each a decoded format ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)+).
- Bundles are **JDLZ/LZC-compressed** ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) — expanded into
  the chunk tree at load ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
- The format is **typed, hierarchical, streamable, and compressible** — extensible (skip unknown chunks) and
  mirroring the asset structure.
- The **bundle-of-chunks is the shipping container** the whole runtime consumes and the whole first half of the
  book decodes — the most foundational data fact.

**Continue:** [C58.4 — The asset pipeline](04-asset-pipeline.md) · [Chapter 58 hub](C58-Build-Pipeline.md)
