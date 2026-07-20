# C36.6 — Loading a Resource End to End

> **The one-sentence version:** loading is: hash the path, resolve it through the layers and memory files to a
> location, read (and decompress if JDLZ) the bundle chunk, and hand the bytes to the format parser — the full
> path from a name to usable data.

[← C36.5 — The A/B/C bundle scheme](05-abc-scheme.md) · [Chapter 36 hub](C36-Archives-VFS.md) ·
[Next: Chapter 37 — The Frame Spine & Engine Modules →](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)

---

## The full load path

Putting the chapter together, loading a resource by path is:

```python
def load_resource(path):
    key = bin_hash(path)                          # C36.3: path → key
    loc = vfs_resolve(key)                        # C36.2/C36.5: highest layer / memory file
    if loc.is_memory_file:                        # C36.4
        raw = memory_file_get(loc)                # served from RAM
    else:
        raw = read_bundle_chunk(loc)              # C36.1: read the bundle's chunk
    if raw[:4] == b"JDLZ":                         # C3.5: compression test
        raw = jdlz_decompress(raw)
    return parse_by_format(raw)                    # hand to the right parser (Ch 5–31)
```

1. **Hash** the path ([C36.3](03-binhash.md)).
2. **Resolve** through the layers ([C36.5](05-abc-scheme.md)) and memory-file manifests
   ([C36.4](04-memoryfile.md)) to a location.
3. **Read** — from RAM (memory file) or the disk bundle chunk ([C36.1](01-bun-model.md)).
4. **Decompress** if JDLZ ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)).
5. **Parse** by format — the EAGL chunk / TPK / SolidList / vault / etc. parser
   ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)).

So the VFS gets you from a path to raw bytes; the format chapters get you from bytes to meaning.

## The loader is StreamMgr

The resource loader is the **StreamMgr singleton at `[0x91A098]`** (verified, 30 references) — the
[resource-streaming manager](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md) that owns
the loaded-resource list. Public forwarders wrap it for call sites:

- **`Stream_FindSection` (`0x507E40`)** — resolve/find a resource.
- **`Stream_AcquireResources` (`0x5033C0`)** / **`Stream_ReleaseResources` (`0x503360`, 37 callers)** — the
  ubiquitous acquire/release for a set of assets.
- **`Stream_BlockUntilLoaded` (`0x503380`)** — wait for a resource, pumping deferred callbacks
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).

So loading isn't a bare file read — it goes through the streaming manager, which handles residency, refcounting,
and layering ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)). The VFS
resolves *where*; StreamMgr manages *when and how long* ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).

> ✅ *Verified:* the loader is StreamMgr `[0x91A098]` with the public forwarders above (addresses and caller
> counts confirmed); bundles are EAGL chunks ([C36.1](01-bun-model.md)); JDLZ decompression is tested first
> ([C3.5](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)).

## Where this connects the book

The load path is the seam between the two halves of the encyclopedia:

- **Before the seam** — the VFS and bundles (this chapter): path → bytes.
- **After the seam** — the formats (Parts I–VI): bytes → meaning.
- **The runtime** (this part) — the class system that turns meaning into behaviour
  ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)).

So a car appears in the world by: the VFS loading its bundle (this chapter) → the geometry/texture parsers
reading it ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[9](../C9-Meshes-FVF/C9-Meshes-FVF.md)) → the
class system instantiating a vehicle object ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md))
→ the frame loop updating it ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)). The loader
is step one.

## Editing implications

- **Edits are found by path-hash** — your edited resource must be at a path the VFS resolves to
  ([C36.2](02-vfs.md), [C36.5](05-abc-scheme.md)).
- **Mind the layers** — a base edit is overridden by a higher layer providing the same path
  ([C36.5](05-abc-scheme.md)).
- **Mind memory files** — a memory-file resource ([C36.4](04-memoryfile.md)) is served from RAM; editing the disk
  copy may not take effect if it's memory-resident.
- **Recompress JDLZ** for compressed bundles ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)).

---

### Key takeaways

- Loading: **hash path → resolve (layers/memory) → read (RAM or bundle chunk) → decompress JDLZ → parse by
  format**.
- The loader is **StreamMgr `[0x91A098]`** with public forwarders (`Stream_FindSection`/`Acquire`/`Release`/
  `BlockUntilLoaded`).
- The VFS resolves *where*; StreamMgr manages *when/how long*
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
- The load path is the **seam**: VFS/bundles (path→bytes) → formats (bytes→meaning) → classes (meaning→
  behaviour).
- Edits must be at a resolvable path, mind layer overrides and memory files, and recompress JDLZ.

**Continue:** [Chapter 37 — The Frame Spine & Engine Modules](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md) ·
[Chapter 36 hub](C36-Archives-VFS.md)
