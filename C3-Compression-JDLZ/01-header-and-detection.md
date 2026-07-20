# C3.1 — The Compression Header & Scheme Detection

> **The one-sentence version:** a 16-byte header names the compression scheme and carries the
> decompressed size; you detect by the four-byte magic, allocate from `decompSize`, and you do it before
> anything tries to read a chunk header.

[← Chapter 3 hub](C3-Compression-JDLZ.md) · [Next: C3.2 — The two flag streams →](02-flag-streams.md)

---

## What it is

Every compressed buffer begins with a fixed 16-byte header. The first four bytes are a FourCC that
selects the scheme:

| Magic | Scheme | Handling |
|---|---|---|
| `'JDLZ'` | the LZ77 variant this chapter documents | full codec (C3.2–C3.4) |
| `'RAWW'` | stored — payload is uncompressed after the header | `data[16:]` verbatim |
| `'HUFF'` | Huffman | rare in this data set; not covered |
| `'COMP'` | composite | rare; not covered |

The JDLZ header layout, confirmed against the retail bundles:

```
+0x00  char[4]  magic       'JDLZ'   bytes 4A 44 4C 5A
+0x04  u8       version     0x02
+0x05  u8       headerSize  0x10     (= 16; where the compressed stream starts)
+0x06  u16      reserved    0x0000
+0x08  u32      decompSize   decompressed output length
+0x0C  u32      compSize     total compressed file length
```

Read on `GLOBAL/GlobalB.lzc` these fields are: version `2`, headerSize `16`, `decompSize` `2,803,648`,
`compSize` `1,520,744` — and the file on disk is exactly `compSize` bytes, which is your first
consistency check.

## How to detect

Detection is a magic comparison, nothing more:

```python
def is_compressed(buf):
    return buf[:4] in (b'JDLZ', b'RAWW', b'HUFF', b'COMP')

def decompressed_size(buf):
    import struct
    return struct.unpack_from('<I', buf, 8)[0]     # +0x08
```

For `'RAWW'`, "decompression" is `buf[16:]`. For `'JDLZ'`, you hand the whole buffer to the decompressor
of [C3.4](04-decompressor.md), which reads `decompSize` to allocate its output and then consumes the
stream starting at `headerSize`.

## Why detection must come first

This is the compression-boundary rule of [C1.8](../C1-EAGL-Container-Model/08-compression-boundary.md),
stated concretely: the `JDLZ` magic sits at offset 0, exactly where a chunk walker would try to read an
`{id, size}` header. If you skip the magic test and walk, you read `4A 44 4C 5A` as a chunk id and the
next four bytes as a size, both nonsense, and desync immediately. The only correct order is **decompress
first, identify second, walk third.** After decompression you re-run the whole identification tree on the
result, because a compressed file could expand to a vault, a chunk tree, or anything else
([C1.7](../C1-EAGL-Container-Model/07-non-chunk-containers.md)).

## Why a header at all

The `decompSize` field is the load-bearing one. A streaming LZ77 decoder *could* grow its output buffer
dynamically, but the engine loads into pre-sized buffers so it can point live structures at the bytes
([C1.12](../C1-EAGL-Container-Model/12-runtime-view.md)). Storing the decompressed size up front lets the
loader allocate the exact output once, with no reallocation and no guessing — which is both faster and
compatible with in-place residency. `compSize` lets a reader validate the file is complete before it
starts. The header is small, fixed, and everything the decoder needs to run without surprises.

## Bending it — safe habits at the boundary

- **Sanity-bound `decompSize` before allocating.** A corrupt header could ask for gigabytes; clamp to a
  sane ceiling (say ≤ 1 GiB) and reject anything larger as corruption rather than trying to allocate it.
- **Cross-check `compSize` against the file length.** A mismatch means truncation or the wrong file;
  catch it here, not fifty chunks deep.
- **Treat `RAWW` as a first-class case.** It is trivial (strip 16 bytes) but real; a reader that only
  handles `JDLZ` will mis-handle a stored buffer.
- **Re-identify after decompressing, every time.** The decompressed bytes are a fresh file; run the magic
  and chunk-header tests on them, don't assume they're a chunk tree.

---

**Continue:** [C3.2 — The two flag streams](02-flag-streams.md) · [Chapter 3 hub](C3-Compression-JDLZ.md)
