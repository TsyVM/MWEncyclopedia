# C58.4 — The Asset Pipeline

> **The one-sentence version:** the content pipeline runs authoring → pack → compress → stream — artists and
> designers author assets in tools, which are packed into `BCHUNK` chunks, compressed (JDLZ/LZC), and streamed at
> runtime — the chain that turns authored content into the shipped, streamed game.

[← C58.3 — The bundle pipeline](03-bundle-pipeline.md) · [Chapter 58 hub](C58-Build-Pipeline.md) ·
[Next: C58.5 — Reading the build & the book's method →](05-reading-build.md)

---

## The pipeline stages

Behind the shipped bundles ([C58.3](03-bundle-pipeline.md)) is a **content pipeline** — the sequence that turns
what artists and designers *make* into what the game *loads*:

```
1. AUTHOR   — artists/designers create assets in tools
              (models, textures, the vault, the world, events)
   ↓
2. PACK     — assets are packed into BCHUNK chunks (C58.3), assembled into bundles
   ↓
3. COMPRESS — bundles are JDLZ/LZC-compressed (Ch 3)
   ↓
4. SHIP     — the compressed bundles go on the disc
   ↓
5. STREAM   — at runtime, bundles are streamed (Ch 38), decompressed, and parsed into the chunk tree
```

So the pipeline is a *one-way flow* from human authoring to machine loading. The book decoded the *output* end
(the bundles, [C58.3](03-bundle-pipeline.md), and how they're loaded,
[Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)); this page traces it back to
the *authoring* end — how the content was made and prepared.

> 🟡 *Reasoned:* the authoring→pack→compress→stream pipeline is the standard game content pipeline, inferred from
> the verified shipping format (bundles/chunks, [C58.3](03-bundle-pipeline.md)), compression
> ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)), and streaming
> ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)); the exact authoring
> tools are outside the shipped executable. The output-side format is verified throughout the book.

## Authoring: the human side

The pipeline begins with **authoring** — the human-created content, in the tools of each discipline:

- **Artists** model the cars ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) and world
  ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) in 3D packages, paint textures
  ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)), and place scenery
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)).
- **Designers** author the vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — car tuning,
  pursuit balance ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), events
  ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) — as *data*.
- **Audio designers** author the banks ([Chapter 19](../C19-Audio-Banks/C19-Audio-Banks.md)), engine sounds
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)), and music
  ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)).
- **The road network** ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) and NIS cutscenes
  ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)) are authored too.

This is where the *data-over-code* architecture ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md))
pays off at the *production* level: because the game is data-driven, *designers author data*, not code. A car's
tuning ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), a pursuit's difficulty
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), a surface's grip
([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) — all authored as data by designers, iterated without
engineering. This is the *reason* the whole engine is data-driven: to let a large team author content in parallel,
each discipline in its own tools, feeding one pipeline.

## Packing: data to bundles

The **pack** stage converts authored assets into the shipping format
([C58.3](03-bundle-pipeline.md)):

- **Each asset becomes chunks** — a model becomes SolidList chunks
  ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), a texture set becomes TPK chunks
  ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), the vault becomes VPAK chunks
  ([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)).
- **The packer mints the hashes** — the asset-hash keys ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) and the
  reflection hashes ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) are computed *at pack time*
  from the authored names, so the runtime looks up by hash
  ([C50.3](../C50-Verification-Methodology/03-hash-verification.md)).
- **Chunks assemble into bundles** — related assets are grouped into bundles
  ([C58.3](03-bundle-pipeline.md)) by scope (global, per-region, per-car).

So packing is where *authored names become runtime hashes* ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)) —
the packer is what turns the human-readable authoring (a car named "EngineRacer",
[C41.3](../C41-Physics-RigidBody/03-hash-unification.md)) into the hashed keys the engine uses. This resolves a
recurring book finding: the *packer mints the hashes* ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) — the
asset-hash scheme is a *pack-time* algorithm, which is why it's deterministic but non-standard
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)). The pack stage is the bridge from authoring (names) to runtime
(hashes and chunks).

## Compress and ship

The final stages ([C58.3](03-bundle-pipeline.md)):

- **Compress** — the assembled bundles are JDLZ/LZC-compressed
  ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) to fit the disc and reduce load times.
- **Ship** — the compressed bundles go on the DVD alongside `speed.exe` ([C58.1](01-shipping-exe.md)).
- **Stream** — at runtime, the streaming manager
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) loads and decompresses
  bundles on demand, parsing the chunks into live objects
  ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)).

So the pipeline ends where the runtime begins: the compressed bundles on the disc are the *interface* between
development and play. Everything the book decoded at runtime — the objects
([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)), the cars
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), the world
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — was *authored, packed, compressed, and shipped*
through this pipeline. Reading the pipeline closes the loop: the game the player runs is the *output* of this
chain, and reverse-engineering it is running the chain *backward* — from the shipped bundles to the authored
intent.

## RE implications

- **The content pipeline** is authoring → pack → compress → stream — human content to loaded objects.
- **Authoring is data** — designers author the vault/events; the data-over-code architecture enables parallel,
  no-recompile production.
- **Packing mints the hashes** — authored names become runtime hashes at pack time
  ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md), the packer-minted asset hash).
- **Compress and ship** — bundles compressed onto the disc, streamed and decompressed at runtime.

---

### Key takeaways

- The **content pipeline** is **authoring → pack → compress → stream** — turning human-created assets into the
  shipped, loaded game.
- **Authoring is data-driven** — artists model/texture, designers author the vault/events/tuning — enabled by the
  data-over-code architecture ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) so a large team
  works in parallel without recompiling.
- **Packing converts assets to chunks and mints the hashes** — authored *names* become runtime *hashes* at pack
  time (resolving why the asset hash is deterministic but non-standard,
  [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)).
- **Compress and ship** — bundles are JDLZ/LZC-compressed onto the disc, then **streamed and decompressed** at
  runtime into live objects.
- Reverse-engineering the game is running this pipeline **backward** — from shipped bundles to authored intent — the
  loop the whole book closes.

**Continue:** [C58.5 — Reading the build & the book's method](05-reading-build.md) · [Chapter 58 hub](C58-Build-Pipeline.md)
