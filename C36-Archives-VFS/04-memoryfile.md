# C36.4 — MemoryFile Intercept

> **The one-sentence version:** some files are served from RAM — a `MemoryFile` manifest (marked by the sentinel
> `0x53219999` at offset 8 of the `*MemoryFile.bin` files) intercepts VFS requests for named resources and
> returns the in-memory copy instead of reading disk.

[← C36.3 — The BinHash](03-binhash.md) · [Chapter 36 hub](C36-Archives-VFS.md) ·
[Next: C36.5 — The A/B/C bundle scheme →](05-abc-scheme.md)

---

## Serving files from RAM

Not every resource is read from disk on demand. The **`MemoryFile`** system serves certain files from **RAM**: a
manifest of memory-resident files intercepts VFS lookups ([C36.2](02-vfs.md)) for those paths and returns the
in-memory copy, bypassing disk. So a request for a memory-file resource is satisfied instantly from RAM rather
than a disk read.

The manifests are the `*MemoryFile.bin` files, and — verified — each carries the sentinel **`0x53219999`** at
offset **8**:

```
GlobalMemoryFile.bin   : 0x53219999 @ 0x8   (a MemoryFile manifest)
PermanentMemoryFile.bin: 0x53219999 @ 0x8
InGameMemoryFile.bin   : 0x53219999 @ 0x8
```

## The sentinel marks a manifest, not content

A critical distinction, verified: **`0x53219999` marks a MemoryFile *manifest*, not a content bundle.**

- The **manifest** (`*MemoryFile.bin`) has the `0x53219999` sentinel and *lists* the memory-resident files.
- The **content** (`GlobalB.lzc`, `DYNTEX.BIN`) does **not** have the sentinel — verified, `GlobalB.lzc` has zero
  occurrences. The content is the actual bundle ([C36.1](01-bun-model.md)); the manifest just names it.

So treating a file with `0x53219999` as a content bundle mis-reads it — it's a manifest of *which* files are
memory-resident, not the files themselves. This is the chapter's parse warning: the sentinel is a manifest
marker, and manifest and content are different files.

> ✅ *Verified:* `0x53219999` appears at offset 8 of the `*MemoryFile.bin` manifests (Global/Permanent/InGame),
> and **not** in the content files (`GlobalB.lzc`); the sentinel marks a manifest, not a bundle.

## Why serve from memory

Serving some files from RAM rather than disk is a performance choice for hot resources:

- **Instant access.** A memory-resident file needs no disk read — the request returns immediately, avoiding a
  seek/read stall.
- **Frequently-used data.** Global data needed constantly (the global bundle, dynamic textures) is worth keeping
  resident.
- **Load-phase residency.** Files loaded once at a phase (permanent, in-game) stay in memory for that phase — the
  `Permanent`/`InGame` manifests name per-phase residency sets.

So the MemoryFile system is a **cache tier** in the VFS: the hottest resources live in RAM (per manifest), and
the rest stream from disk ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
The manifests decide which files get the RAM tier.

## The three manifests

The three `*MemoryFile.bin` manifests correspond to residency scopes:

- **`GlobalMemoryFile.bin`** — global resources resident throughout.
- **`PermanentMemoryFile.bin`** — permanently-resident data (small — 3200 bytes; a core set).
- **`InGameMemoryFile.bin`** — in-game-phase resources, resident during gameplay.

So residency is scoped by phase ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)):
what's memory-resident depends on the game phase, and the matching manifest names it. The VFS consults the active
manifests to decide whether a request hits RAM or disk.

## RE implications

- **`0x53219999` marks a MemoryFile manifest** — a list of memory-resident files, **not** a content bundle.
- **Manifest ≠ content** — the manifest (`*MemoryFile.bin`) names files; the content (`GlobalB.lzc`) is elsewhere.
- **Memory files bypass disk** — the VFS serves them from RAM ([C36.2](02-vfs.md)).
- **Residency is phase-scoped** — Global/Permanent/InGame manifests name per-scope resident sets
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).

---

### Key takeaways

- **`MemoryFile`** serves resources from RAM; a manifest intercepts VFS requests for named files.
- Manifests (`*MemoryFile.bin`) carry the sentinel **`0x53219999`** at offset 8 (verified) — and it marks a
  **manifest, not a bundle**.
- The **content** files (`GlobalB.lzc`, `DYNTEX.BIN`) lack the sentinel — manifest and content are distinct.
- Serving from memory is a **cache tier** for hot/frequently-used resources, avoiding disk stalls.
- Three manifests scope residency by phase: **Global, Permanent, InGame**.

**Continue:** [C36.5 — The A/B/C bundle scheme](05-abc-scheme.md) · [Chapter 36 hub](C36-Archives-VFS.md)
