# C3.6 — Where JDLZ Lives, and What It Doesn't Tell You

> **The one-sentence version:** JDLZ wraps the large static bundles, the per-texture blobs in compressed
> packs, and every minimap tile — but compression is a content-agnostic wrapper, so a `.lzc` tells you
> nothing about what's inside until you decompress, and not everything inside is a flat chunk tree.

[← C3.5 — Writing compressed data back](05-writing-compressed-back.md) · [Chapter 3 hub](C3-Compression-JDLZ.md) ·
[Next chapter: C4 — Byte-Level Toolcraft →](../C4-Byte-Level-Toolcraft/C4-Byte-Level-Toolcraft.md)

---

## Where you'll meet it

| Data | Why it's compressed | Covered in |
|---|---|---|
| Global / front-end / in-game bundles (`GlobalB.lzc`, `FrontB.lzc`, `InGameB.lzc`, `gameplay.lzc`) | large, static, read once at load; compression saves disk and load bandwidth | C15, C27, C36 |
| Per-texture blobs inside a compressed TPK | many shared car-template textures; each compressed independently | [C5](../C5-Textures-TPK/C5-Textures-TPK.md) |
| Each minimap tile | many small textures, individually compressed and decompressed on demand | [C29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md) |

The common thread is *bulk, static, or numerous* data where compression pays. Hot per-frame data is not
compressed — decompression cost would defeat the purpose.

## Compression is a wrapper, not a format

The most important conceptual point in this whole chapter: **`JDLZ` says how the bytes are packed, not
what they are.** A compressed vault, a compressed chunk tree, and a compressed texture blob all begin
with the same `4A 44 4C 5A`. You cannot tell them apart from the outside; you decompress first and
identify second ([C1.7](../C1-EAGL-Container-Model/07-non-chunk-containers.md),
[C1.8](../C1-EAGL-Container-Model/08-compression-boundary.md)). This is why the universal opener puts the
compression test *before* the magic branch: the real magic is hidden underneath.

A concrete consequence for tooling: never key behaviour off the `.lzc` extension or the `JDLZ` magic
beyond "decompress it." The moment you assume "`.lzc` means fonts" or "`JDLZ` means a chunk tree," you
will meet the counterexample. Decompress, then run the full identification tree on the result.

## A caution from the data: not every bundle is a flat chunk tree

Decompressing the four global bundles makes the point vividly. Three of them (`GlobalB`, `FrontB`,
`InGameB`) expand and then walk cleanly as flat chunk trees from the first byte to the last — 194, 274,
and 75 top-level chunks respectively, every chunk's `8 + size` landing exactly on the next header. The
fourth, `gameplay.lzc`, expands to exactly its header's `decompSize` (2,105,216 bytes — the decompressor
is unquestionably correct) but a naive flat chunk walk stops early after a handful of chunks.

That is not a decompression failure; it is a *content* fact. The decompressed `gameplay` bundle is not a
single flat chunk stream — its structure is more involved, and reading it belongs to the chapter that
owns that data, not to the codec. The lesson generalises: **a correct decompressor guarantees the bytes,
not their shape.** Verify decompression by the exact `decompSize` match (which is unambiguous), and treat
"does it walk as chunks?" as a separate question answered by the appropriate format chapter. Conflating
the two — "my walk desynced, so my decompressor is broken" — sends you debugging the wrong layer.

## Why compression is a separate outer layer

Making compression a uniform, content-agnostic wrapper with its own magic buys the engine one
decompressor shared across every subsystem: the texture loader, the vault loader, and the world loader
all call the same JDLZ path and then proceed as if the bytes had been uncompressed on disk. The content
format never changes — a decompressed bundle is byte-identical to what an uncompressed one would have
been — so nothing downstream needs to know compression ever happened. It is the same separation-of-concerns
that the container bit gives structure ([C1.1](../C1-EAGL-Container-Model/01-the-container-bit.md)):
one orthogonal mechanism, reused everywhere, with no special cases.

## Bending it — practical rules

- **Decompress, then identify — always in that order.** The `.lzc`/`JDLZ` surface is opaque by design.
- **Trust the size match, not the walk, for codec correctness.** `len(output) == decompSize` proves the
  decompressor; the walk proves nothing about the codec.
- **Route decompressed content to its owning chapter.** Once expanded, a bundle is just a file — send it
  through the normal identification tree and hand each part to the format that owns it.
- **Prefer uncompressed re-saves for edited bundles** where the loader allows, keeping the original `.lzc`
  as backup ([C3.5](05-writing-compressed-back.md)).

---

### Chapter 3 — where you've arrived

You can detect compression, decompress any `.lzc` in the game with a codec proven against the retail
bundles, explain every bit of the two-stream, near/far design, and write data back without corrupting it.
Combined with the container model of C1 and the identifiers of C2, you now have the three primitives —
*structure, names, packing* — that every remaining chapter assumes.

**Back to:** [Chapter 3 hub](C3-Compression-JDLZ.md) ·
**Next chapter:** [C4 — Byte-Level Toolcraft](../C4-Byte-Level-Toolcraft/C4-Byte-Level-Toolcraft.md)
