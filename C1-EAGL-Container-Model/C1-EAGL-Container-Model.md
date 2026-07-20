# Chapter 1 — The EAGL Container Model

> **Goal of this chapter:** after reading it you can open *any* file in the game, walk its structure,
> and know whether a given blob is "more structure" or "raw data" — without a parser written for that
> specific file. You will also be able to edit one safely and write it back so the game still loads
> it. Everything else in the encyclopedia builds on this one idea.

The engine stores almost all of its data as a **chunk tree**. A file is not a bespoke format with a
hand-written header for every asset type; it is a recursive sequence of self-describing blocks. Master
this single concept and the game stops being a wall of binary and becomes a navigable hierarchy that
you can dump, diff, edit, and rebuild with a dozen lines of code that never change from one format to
the next.

This chapter is the foundation. It is deliberately exhaustive, because every later chapter assumes it
completely: when C8 says "the geometry object header is a leaf inside the `GeometryObject` container,"
it is relying on you already knowing what "leaf," "container," and "inside" mean at the byte level.

---

## Deep-dive pages

The overview below is the map. Each core mechanism then gets a focused page covering **what it is, how
it works, why it's built that way, and what happens if you bend it** — the right way or the wrong way.

- [C1.1 — The Container Bit (`0x80000000`)](01-the-container-bit.md): the one flag that makes files self-describing.
- [C1.2 — The Chunk Header & the Off-by-Eight](02-chunk-header-and-sizes.md): why `size` excludes the header and how the size tree must balance.
- [C1.3 — Walking the Tree: Iteration, Recursion & Safe Readers](03-walking-the-tree.md): the algorithms, in C and Python, with bounds checking.
- [C1.4 — Alignment Padding (`0x11`) & Null Chunks](04-alignment-and-padding.md): the filler the engine inserts, and the one catch when you touch it.
- [C1.5 — Endianness Islands](05-endianness-islands.md): little-endian everywhere except the EA audio stream headers.
- [C1.6 — Matrices & the Z-up Coordinate System](06-matrices-and-coordinates.md): read straight, no transpose, convert axes only at the Y-up boundary.
- [C1.7 — The Non-Chunk Container Models](07-non-chunk-containers.md): VPAK, EA audio, DDS — the files that are *not* chunk trees, and how to branch early.
- [C1.8 — The Compression Boundary](08-compression-boundary.md): why every opener must test for JDLZ before it does anything else.
- [C1.9 — Building a Universal Opener & Dumper](09-universal-opener.md): a complete, portable tool you will reuse in every chapter.
- [C1.10 — The Size Tree in Practice: Editing & Repacking](10-editing-and-repacking.md): in-place edits, ancestor fixups, and atomic writes.
- [C1.11 — Failure Modes & Forensics](11-failure-modes.md): how to read a desynced walk and find the byte that broke it.
- [C1.12 — The Runtime View: How the Engine Walks the Same Tree](12-runtime-view.md): the loader, the handler registry, and streaming-in-place.

---

## 1.1 The chunk header

Every chunk begins with an 8-byte, little-endian header:

```c
struct ChunkHeader {
    uint32_t id;    // chunk type identifier (top bit = container flag)
    uint32_t size;  // length of the payload in bytes, EXCLUDING these 8 header bytes
};
```

The bytes that follow — exactly `size` of them — are the **payload**. Immediately after the payload
comes the next chunk's header. A file is therefore just:

```
[id|size][payload ............][id|size][payload ...] ...
```

repeated until the buffer is consumed. There is no file-level header, no table of contents, no
directory: the structure *is* the sequence of headers. This is what makes the format so uniform to
parse — the same eight-byte read starts every chunk in the game.

## 1.2 The container bit

The single most important rule in the whole engine:

> If **bit 31** of `id` is set (`id & 0x80000000`), the chunk is a **container**: its payload is
> itself a sequence of chunks. Otherwise the chunk is a **leaf**: its payload is raw data.

```c
static inline int is_container(uint32_t id) { return (id & 0x80000000u) != 0; }
```

This is why you can navigate unknown files safely. You never have to guess whether to recurse — the
id tells you. `0x80134000` (GeometryContainer) is a container; `0x00134011` (an object header) is a
leaf. Note that they share the same family digits (`_0134___`) and differ only in the top bit: the
container bit is a *role* flag riding on top of a *type*. Chapter [C1.1](01-the-container-bit.md)
takes this apart in full.

## 1.3 Sizes exclude the header

`size` counts payload bytes only. To advance to the next sibling chunk you step `8 + size` bytes, not
`size`. Stepping `size` is the **off-by-eight**: you land eight bytes early, read a mid-payload byte
as the next id, and every chunk after that is rubbish. Because a container's payload *is* its
children, sizes form a strict accounting tree:

```
parent.size  ==  Σ over children ( 8 + child.size )
```

The consequence that bites every modder: **change the length of a leaf payload and every ancestor's
size is now wrong.** Fixing that from the leaf up to the root is the "ancestor-size fixup" that
governs all repacking. See [C1.2](02-chunk-header-and-sizes.md) and, for the workflow,
[C1.10](10-editing-and-repacking.md).

## 1.4 Walking a flat chunk stream

The minimal correct walk, bounds-checked so a malformed file stops cleanly instead of crashing:

```c
// Iterate the top-level chunks of a buffer [data, data+len).
void walk(const uint8_t* data, size_t len) {
    size_t off = 0;
    while (off + 8 <= len) {
        uint32_t id, size;
        memcpy(&id,   data + off,     4);   // little-endian host assumed
        memcpy(&size, data + off + 4, 4);
        const uint8_t* payload = data + off + 8;
        if (off + 8 + (size_t)size > len) break;   // truncated / not a chunk here
        visit(id, payload, size);
        off += 8 + (size_t)size;
    }
}
```

```python
import struct
def walk(buf):
    off = 0
    while off + 8 <= len(buf):
        cid, size = struct.unpack_from('<II', buf, off)
        if off + 8 + size > len(buf): break
        yield cid, off + 8, size          # id, payload offset, payload size
        off += 8 + size
```

To traverse the whole tree, descend whenever the container bit is set. The full recursive walker,
with the absolute-offset bookkeeping you need for patching, is in [C1.3](03-walking-the-tree.md). A
tree-printing "dump" built on it is the most useful twelve lines in your toolbox — run it against an
unknown file and the ids alone tell you what subsystem you are looking at (match them against
[the master table](../Glossary/chunk-ids.md)).

## 1.5 Alignment padding and null chunks

Many payloads are prefixed with a run of the byte `0x11` to align the meaningful data to a boundary.
It carries no information — strip it before parsing. A separate **null chunk** (`id == 0x00000000`)
appears between chunks as filler; step over it like any other chunk. Both are *counted* bytes even
though they are meaningless, so you cannot simply delete them without disturbing the size tree. See
[C1.4](04-alignment-and-padding.md).

## 1.6 Endianness

Everything is **little-endian** unless a chapter says otherwise. The one exception is inside EA audio
stream records (the `PT` chunks of a sound bank and the `SCHl` header fields of a music file), which
are **big-endian**. When a "sample rate" reads as `0x44AC0000` instead of `0x0000AC44` (44100), you
have read the wrong endianness. These narrow big-endian regions are the "islands" of
[C1.5](05-endianness-islands.md).

## 1.7 Matrices and the coordinate system

Transforms are stored as 4×4 float matrices in **D3D row-major** order with the **translation in row
3** (floats 12–14). That memory layout is *identical* to the column-major convention many math
libraries use, so a straight 16-float sequential read is already correct — **do not transpose.** The
world is **Z-up**, and the engine performs no axis conversion on load or save; if you bridge to a
Y-up tool, do the swap in that bridge, never in the parser. See
[C1.6](06-matrices-and-coordinates.md).

## 1.8 Not everything is a chunk tree

Three important container models are *not* chunk trees and must be recognised at file offset 0, before
you try to read a chunk header:

- **`VPAK`** attribute vaults — a completely different container ([C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)).
- **EA audio** streams and banks (`ABKC`/`BNKl`/`SCHl`/`MPFF`) — big-endian TLV records ([C19](../C19-Audio-Banks/C19-Audio-Banks.md)).
- **`DDS `** textures — a fixed 128-byte header ([C6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).

Branch on these magics first. [C1.7](07-non-chunk-containers.md) explains why they exist and how to
tell them apart.

## 1.9 Compression comes first of all

Some files are wrapped in a 16-byte **JDLZ** header (magic `'JDLZ'`) and must be decompressed before
*any* magic or chunk header is visible. Always test for compression before anything else:

```python
def maybe_decompress(buf):
    if buf[:4] == b'JDLZ':
        return jdlz_decompress(buf)   # Chapter 3
    return buf
```

The full algorithm and a complete decompressor are in
[Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md); why it must be the very first test is
[C1.8](08-compression-boundary.md).

## 1.10 Putting it together — a universal opener

```python
def open_eagl(path):
    buf = open(path, 'rb').read()
    if buf[:4] == b'JDLZ':
        buf = jdlz_decompress(buf)                       # Chapter 3
    # Non-chunk container models — branch early:
    if buf[:4] == b'VPAK': return ('vault', buf)         # Chapter 11
    if buf[:4] == b'DDS ': return ('dds',   buf)         # Chapter 6
    if buf[:4] in (b'ABKC', b'BNKl', b'SCHl', b'MPFF'):  # Chapter 19/21
        return ('audio', buf)
    return ('chunks', list(walk_tree(buf)))              # the common case
```

From here, each subsequent chapter takes a specific chunk id (or non-chunk magic) and tells you how to
interpret its payload. The complete, hardened version of this opener — with a dumper, a hex fallback,
and error handling — is built step by step in [C1.9](09-universal-opener.md), and you will carry it
into every other chapter of the book.

## 1.11 The same tree, seen by the engine

Everything above is how *your* tools read the file. The engine reads it almost identically: a generic
loader walks the chunk stream, tests bit 31 to decide whether to recurse, and dispatches each
recognised leaf id to a registered handler — the **chunk handler registry**. Unknown ids are stepped
over harmlessly. Understanding that the game and your dumper are running the *same* algorithm is what
makes reverse engineering tractable: if your walk desyncs, the engine's would too, so a file the game
loads is a file your correct walker can also parse. The runtime side is [C1.12](12-runtime-view.md).

---

## Key takeaways

- A file is `[id|size|payload]` repeated; step `8 + size`, never `size`.
- `id & 0x80000000` ⇒ container (recurse); otherwise leaf (data).
- The size tree must balance: `parent.size == Σ(8 + child.size)`. Fix ancestors after any length change.
- Strip `0x11` padding; skip null chunks; both are counted, not free.
- Little-endian everywhere except EA audio TLV fields (big-endian islands).
- Matrices read straight (no transpose); the world is Z-up, converted only at a Y-up boundary.
- Decompress JDLZ first; VPAK, EA audio, and DDS are separate container models — branch on their magic.
- Your dumper and the engine's loader run the same walk, which is what makes the format learnable.

**Next:** [Chapter 2 — Identifiers & Hashing](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md),
because the names inside these chunks are almost never stored as text.
