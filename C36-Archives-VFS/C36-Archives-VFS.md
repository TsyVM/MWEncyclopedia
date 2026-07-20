# Chapter 36 — Archives & the Virtual File System

> **Goal of this chapter:** decode how the game finds and loads its data — the `.BUN` bundle model (an EAGL
> chunk file), the path→BinHash virtual file system that resolves names to bundles, the `MemoryFile` intercept
> that serves some files from RAM, and the A/B/C bundle-loading scheme.

The formats of Parts I–VI live inside **bundles** (`.BUN`), and the game reaches them through a **virtual file
system**: it asks for a resource by path, the VFS hashes the path to a key and finds it, and the loader serves
it — from a bundle on disk or, sometimes, from a `MemoryFile` in RAM. This chapter decodes that loading
substrate — the layer between "a file exists" and "the game has it."

> **Grounded in verified structure.** A shipped bundle (`.BUN`) is an **EAGL chunk file**
> ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — the same container as everything else,
> with **no `'BUN '` magic** (the extension is a convention). The VFS resolves a **path → BinHash key**. Some
> assets are served from **`MemoryFile`** manifests marked by the sentinel `0x53219999` (e.g. `GlobalB.lzc`,
> `DYNTEX.BIN`) — *not* content bundles. The resource loader is the StreamMgr singleton at `[0x91A098]`
> (verified, 30 references; [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).

---

## Deep-dive pages

- [C36.1 — The .BUN bundle model](01-bun-model.md): a bundle is an EAGL chunk file, extension-not-magic.
- [C36.2 — The virtual file system](02-vfs.md): path → BinHash key → resource.
- [C36.3 — The BinHash](03-binhash.md): the path hash that keys the VFS.
- [C36.4 — MemoryFile intercept](04-memoryfile.md): serving files from RAM, and the `0x53219999` sentinel.
- [C36.5 — The A/B/C bundle scheme](05-abc-scheme.md): the layered bundle loading.
- [C36.6 — Loading a resource end to end](06-loading.md): from a path to bytes in memory.

---

## 36.1 A bundle is an EAGL chunk file

A `.BUN` bundle is not a special archive format — it's an **EAGL chunk file**
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)), the same `{id, size}` chunk tree as a TPK,
a SolidList, or a track file. The `.BUN` extension is a **convention**, not a magic number: there is **no
`'BUN '` FourCC** — a bundle is identified by being an EAGL chunk file at a bundle path, not by a header tag
([C36.1](01-bun-model.md)). So the universal chunk tools ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md))
open a bundle directly; there's no archive layer to peel first.

## 36.2 The VFS: path → key → resource

The game asks for resources by **name/path**, and the **virtual file system** resolves that path to a resource
([C36.2](02-vfs.md)):

```
"GLOBAL\CARS\COBALTSS\GEOMETRY.BIN"  → BinHash(path) → key → find the resource → bytes
```

The path is **hashed** to a key (the BinHash, [C36.3](03-binhash.md)); the VFS looks up that key to find where
the resource lives (a bundle, a memory file); the loader serves it. So the game code references data by
readable path, and the VFS turns that into an actual resource — the "reference resolves to data" pattern
([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)) applied to files.

## 36.3 The BinHash keys everything

The VFS key is the **BinHash** of the path ([C36.3](03-binhash.md)) — a hash that turns a path string into a
32-bit key the loader indexes by. So a resource's identity in the running game is its path-hash, and the VFS is
a map from path-hashes to resources. This is why resources are referenced by hash internally (compact) while
authored by path (readable) — the BinHash bridges the two.

## 36.4 MemoryFile: some files live in RAM

Not every resource is read from disk on demand. The **`MemoryFile`** system serves certain files from **RAM** —
a manifest of memory-resident files intercepts VFS requests for them and returns the in-memory copy
([C36.4](04-memoryfile.md)). These manifests are marked by the sentinel **`0x53219999`** and name files like
`GlobalB.lzc` and `DYNTEX.BIN`. A crucial warning: **`0x53219999` marks a MemoryFile *manifest*, not a content
bundle** — treating the manifest as a bundle mis-reads it. So the VFS has two backends: disk bundles and memory
files, chosen per resource.

## 36.5 The A/B/C scheme

Bundles load in a layered **A/B/C** scheme ([C36.5](05-abc-scheme.md)): the game loads bundles in tiers (a base
"A" layer, then "B", then "C" overrides/additions), so later layers extend or override earlier ones. This is
how the game composes its data — a global base plus per-context layers — and how patches/DLC could add content
without replacing the base. The layering is resolved by the VFS: a path resolves to the highest layer that
provides it.

---

### Key takeaways

- A `.BUN` bundle is an **EAGL chunk file** — the extension is a convention, there's **no `'BUN '` magic**.
- The **VFS** resolves a **path → BinHash key → resource** — code references data by path, the VFS finds it.
- The **BinHash** of the path is the VFS key — resources are hash-identified internally, path-authored
  externally.
- **`MemoryFile`** serves some files from RAM (manifests marked `0x53219999`, e.g. `GlobalB.lzc`) — *not*
  content bundles.
- Bundles load in a layered **A/B/C** scheme; a path resolves to the highest layer providing it.

**Next:** [Chapter 37 — The Frame Spine & Engine Modules](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md):
the loop that drives it all.
