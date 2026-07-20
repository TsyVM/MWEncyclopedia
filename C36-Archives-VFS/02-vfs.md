# C36.2 — The Virtual File System

> **The one-sentence version:** the game asks for resources by path, the VFS hashes the path to a BinHash key
> and looks it up, and the loader serves the resource — from a disk bundle or a memory file — so code
> references data by readable name and the VFS finds it.

[← C36.1 — The .BUN bundle model](01-bun-model.md) · [Chapter 36 hub](C36-Archives-VFS.md) ·
[Next: C36.3 — The BinHash →](03-binhash.md)

---

## Reference by path, resolve by hash

The engine references its data by **path** — `"GLOBAL\CARS\COBALTSS\GEOMETRY.BIN"`, `"TRACKS\STREAML2RA.BUN"` —
and the **virtual file system** turns that path into an actual resource:

```
path → BinHash(path) → key → VFS lookup → resource location → bytes
```

1. **Hash the path** to a key (the BinHash, [C36.3](03-binhash.md)).
2. **Look up the key** in the VFS index → where the resource lives (which bundle/memory file, at what offset).
3. **Serve** the bytes — read from the bundle ([C36.1](01-bun-model.md)) or the memory file
   ([C36.4](04-memoryfile.md)).

So the VFS is a **map from path-hashes to resources**. Code names data readably; the VFS resolves it to bytes —
the file-loading version of the "reference resolves to data" indirection that runs the whole engine
([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)).

## Why a virtual file system

A VFS layer between "code wants a resource" and "bytes on disk" buys the engine flexibility:

- **Location independence.** Code asks for a path; the VFS decides *where* it comes from — a specific bundle, a
  layer ([C36.5](05-abc-scheme.md)), or RAM ([C36.4](04-memoryfile.md)). The code doesn't know or care.
- **Layering/overrides.** The A/B/C scheme ([C36.5](05-abc-scheme.md)) lets a later layer provide a path,
  overriding an earlier one — resolved transparently by the VFS.
- **Interception.** Some paths are served from memory ([C36.4](04-memoryfile.md)) instead of disk, without the
  requesting code changing.
- **Hash-keyed speed.** Resolving a path is a hash lookup ([C36.3](03-binhash.md)), not a directory scan.

So the VFS decouples *what* code wants (a path) from *where* it is (bundle/layer/memory) — the layer that makes
the whole loading system flexible.

## Path-hash as resource identity

Internally, a resource's identity is its **BinHash** ([C36.3](03-binhash.md)) — the hash of its path — not the
path string. So:

- **References are compact** — a 32-bit key, not a path string.
- **Lookups are fast** — hash-indexed ([C8.6](../C8-Geometry-Solids/06-lookup.md)-style).
- **Paths are authored** — humans use readable paths; the VFS hashes them.

This is the same "readable name, hashed key" split as the vault ([C11.2](../C11-Attribute-Vaults/02-erts-strings.md))
and the class registry ([C32.4](../C32-Runtime-Class-System/04-registration.md)): author by name, resolve by
hash. The VFS applies it to files.

> ✅ *Verified:* the VFS resolves a **path → BinHash key**; the resource loader is the StreamMgr singleton
> `[0x91A098]` (verified, [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
> 🟡 *Reasoned:* the exact VFS index structure (hash table of key → location) is per-system RE; the path→hash→
> resource model and the BinHash key are verified/documented.

## Two backends

The VFS serves from two backends, chosen per resource ([C36.4](04-memoryfile.md)):

- **Disk bundles** ([C36.1](01-bun-model.md)) — the default; read (and decompress) the bundle chunk.
- **Memory files** ([C36.4](04-memoryfile.md)) — resources served from RAM via a `MemoryFile` manifest.

So a VFS lookup resolves not just *which resource* but *which backend serves it* — the requesting code gets
bytes either way, transparently.

## RE implications

- **The VFS maps path-hash → resource** — trace a load from a path to its BinHash to its location.
- **Resources are BinHash-keyed** ([C36.3](03-binhash.md)) — compact, fast, name-authored.
- **Two backends** — disk bundles and memory files ([C36.4](04-memoryfile.md)); the VFS picks.
- **The loader is StreamMgr `[0x91A098]`** ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).

---

### Key takeaways

- The **VFS** resolves **path → BinHash key → resource → bytes** — code references by path, the VFS finds it.
- A VFS buys **location independence, layering/overrides, interception, and hash-keyed speed**.
- A resource's internal identity is its **path-hash** (BinHash) — compact, fast, name-authored.
- It's the "readable name, hashed key" split (like the vault and class registry) applied to files.
- The VFS serves from **two backends** (disk bundles, memory files), chosen transparently per resource.

**Continue:** [C36.3 — The BinHash](03-binhash.md) · [Chapter 36 hub](C36-Archives-VFS.md)
