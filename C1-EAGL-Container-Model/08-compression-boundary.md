# C1.8 — The Compression Boundary

> **The one-sentence version:** JDLZ compression is a wrapper that hides every other magic and every
> chunk header, so testing for it must be the very first thing any opener does — before identification,
> before walking, before anything.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.7 — Non-chunk containers](07-non-chunk-containers.md) ·
[Next: C1.9 — Universal opener →](09-universal-opener.md)

---

## What it is

Some files are wrapped in a 16-byte **JDLZ** header (magic `'JDLZ'` at offset 0) followed by a
compressed byte stream. Until you decompress it, the file's *real* identity is invisible: a compressed
texture pack, a compressed vault, and a compressed world bundle all look identical from outside — they
all start `4A 44 4C 5A` (`JDLZ`). Compression is therefore a **boundary**, a layer you must cross before
any of the identification logic from [C1.7](07-non-chunk-containers.md) applies.

```python
def maybe_decompress(buf):
    if buf[:4] == b'JDLZ':
        return jdlz_decompress(buf)   # full algorithm in Chapter 3
    return buf
```

The 16-byte header carries the magic plus the sizes the decompressor needs (notably the uncompressed
length, so it can allocate the output buffer up front). The full byte layout and a complete,
round-trippable decompressor and compressor are in
[Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md); this page is only about *where in your
pipeline the test belongs* and *why it must come first*.

## How it fits the pipeline

The correct order of operations for opening any file is:

1. **Test for `JDLZ`.** If present, decompress, and **restart from step 1** on the decompressed bytes
   (in principle a stream could be double-wrapped; in practice it is one layer, but restarting costs
   nothing and is robust).
2. **Test the non-chunk magics** (`VPAK`, `DDS `, `ABKC`/`BNKl`, `SCHl`, `MPFF`, `LOCH`, `MZ`) —
   [C1.7](07-non-chunk-containers.md).
3. **Otherwise, read a chunk header and walk** — [C1.3](03-walking-the-tree.md).

The single most common beginner mistake is doing this in the wrong order — reading the first four bytes,
seeing they aren't a chunk id you recognise, and concluding "unknown format," when in fact the file is a
perfectly ordinary bundle that simply happens to be compressed. The `JDLZ` magic is *right there* at
offset 0; the fix is to test for it before you test for anything else.

## Why it is designed this way

Compression is orthogonal to content. EA compresses whatever is worth compressing — large global
bundles, font packs, minimap tiles — regardless of what container model the *content* uses. Making
compression a uniform outer wrapper with its own magic means:

- **One decompressor serves every subsystem.** The texture loader, the vault loader, and the world
  loader all call the same JDLZ path and then proceed as if the bytes had been uncompressed on disk.
- **The content format doesn't change.** A compressed vault is byte-for-byte identical to an
  uncompressed vault once expanded; nothing downstream needs to know it was compressed. The wrapper is
  invisible past the boundary.

> 🟡 *Reasoned:* treating compression as a content-agnostic outer layer is the natural reading of how
> `JDLZ` appears across unrelated file types; the *fact* that these specific files are JDLZ-wrapped is
> ✅ verified, and the decompressor round-trips them.

Note also the interaction with minimap tiles, which are `JDLZ`-wrapped TPK tiles ([C29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)):
they are a concrete example of the boundary — the outer bytes say `JDLZ`, and only after decompression
does the inner `0x0003A100` / TPK structure appear.

## Bending it — do and don't

- **Do decompress first, always.** Build the `maybe_decompress` call into the very front of your opener
  so you can never forget it. Every worked example in this book assumes the bytes it receives are
  already decompressed.
- **Do re-identify after decompressing.** The output of `jdlz_decompress` is a fresh file that needs the
  full identification tree run on it — it might be a vault, a chunk tree, or anything else.
- **Don't assume a `.LZC` extension means compressed and nothing else means uncompressed.** The
  extension correlates but does not guarantee; the `JDLZ` magic is the only reliable test. Some `.BIN`
  and `.BUN` files are compressed and some `.LZC`-adjacent data is not — trust the bytes.
- **Don't leave a file half-compressed after editing.** If you decompress, edit, and write back, either
  re-compress with a correct JDLZ header (Chapter 3) or save uncompressed *only if* the game will accept
  it for that asset. Mismatching the wrapper and the content is a guaranteed load failure.

---

**Continue:** [C1.9 — Building a universal opener & dumper](09-universal-opener.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
