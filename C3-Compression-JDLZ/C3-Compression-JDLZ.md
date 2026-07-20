# Chapter 3 — Compression (JDLZ / `.lzc`)

> **Goal of this chapter:** detect a compressed file, decompress it correctly with a complete algorithm
> verified against the retail bundles, understand *why* JDLZ is built the way it is, and know your
> options — and their trade-offs — for writing compressed data back.

A meaningful fraction of the game's data is wrapped in EA Black Box's **JDLZ** compression: the large
global and front-end bundles (`GlobalB.lzc`, `FrontB.lzc`, `InGameB.lzc`, `gameplay.lzc`), the
per-texture blobs inside compressed texture packs, and every minimap tile. Until you decompress one, no
chunk header is visible — the `JDLZ` magic hides everything downstream ([C1.8](../C1-EAGL-Container-Model/08-compression-boundary.md)).
This chapter gives you a working, verified codec and the judgement to use it safely.

> **Verified against retail data.** The decompressor below expands `GLOBAL/GlobalB.lzc` (1,520,744 bytes
> in) to exactly **2,803,648 bytes** — the size written in its own header — and the result walks as a
> clean tree of 194 top-level chunks. The same codec matches the header size on all four global bundles.
> Every formula on the following pages is confirmed by that round trip, not merely plausible.

---

## Deep-dive pages

- [C3.1 — The compression header & scheme detection](01-header-and-detection.md): the 16-byte header, the FourCC family (`JDLZ`/`RAWW`/…), and why detection must precede any chunk walk.
- [C3.2 — The two flag streams](02-flag-streams.md): `flags1`/`flags2`, the `0x100` sentinel refill trick, and why splitting the two decisions is the clever core of the format.
- [C3.3 — Near vs far back-references](03-backreferences-near-far.md): the two encodings, the exact bit-packing math, and why overlapping copies are how runs get stored.
- [C3.4 — The complete decompressor](04-decompressor.md): the verified algorithm in Python and C, with the robustness guards a production tool needs.
- [C3.5 — Writing compressed data back](05-writing-compressed-back.md): uncompressed vs. literal-only vs. real compression, and the round-trip discipline that keeps you safe.
- [C3.6 — Where JDLZ lives, and what it doesn't tell you](06-where-jdlz-lives.md): the real files, the compression-is-a-wrapper principle, and a caution from a bundle that *isn't* a flat chunk tree.

---

## 3.1 The shape of the format

JDLZ is a byte-oriented **LZ77** variant. Like all LZ77 it encodes a stream as a mix of two things:

- **Literals** — "emit this raw byte" — for data it has not seen before.
- **Back-references** — "copy `length` bytes from `distance` bytes ago in the output" — for data that
  repeats.

What makes JDLZ *JDLZ*, and distinguishes it from a textbook LZ77, is how it signals those decisions: it
carries **two independent 1-bit flag streams**, refilled a byte at a time and tracked with a neat `0x100`
sentinel. One stream answers "literal or reference?"; the other answers "if a reference, near or far?".
Separating the two questions lets each stream stay well-packed, and it is the single design idea worth
understanding before you read a byte. It is unpacked in [C3.2](02-flag-streams.md).

## 3.2 The header, at a glance

A compressed buffer opens with a 16-byte header. The bytes you actually need are the magic (to detect)
and `decompSize` at offset `+0x08` (to size your output buffer):

```
+0x00  char[4]  magic       'JDLZ'   (4A 44 4C 5A)
+0x04  u8       version     0x02
+0x05  u8       headerSize  0x10
+0x06  u16      reserved    0
+0x08  u32      decompSize   decompressed length
+0x0C  u32      compSize     total compressed length (whole file)
```

Detection is a magic test, and it must happen before anything tries to read a chunk header
([C1.8](../C1-EAGL-Container-Model/08-compression-boundary.md)). The full header treatment, the sibling
`RAWW` "stored" scheme, and why `decompSize` must be sanity-bounded before you allocate are in
[C3.1](01-header-and-detection.md).

## 3.3 Decoding, in one paragraph

Starting just past the header, you consume the input while producing output: refill each flag stream when
its sentinel says it is empty; read one `flags1` bit — if 0, copy one literal byte; if 1, read one
`flags2` bit and two payload bytes and perform a **near** or **far** back-reference copy from earlier in
the output. Repeat until the output reaches `decompSize`. The two back-reference encodings pack length and
distance into their two bytes differently ([C3.3](03-backreferences-near-far.md)), and the copy is allowed
to *overlap* the write cursor — that overlap is precisely how a run of repeated bytes is encoded. The
complete loop, ready to paste, is [C3.4](04-decompressor.md).

## 3.4 Writing it back — the honest summary

There is no widely-used general-purpose JDLZ *compressor*, and most of the time you do not need one:

- When you rebuild a bundle that was `.lzc`, you can usually write it back **uncompressed** — the loader
  reads a raw chunk tree fine when the `JDLZ` magic is absent. The file grows; it loads.
- When a container *requires* a JDLZ payload (a minimap tile the loader expects to be compressed), you can
  emit a **literal-only** JDLZ stream: valid and game-readable, just larger than optimal.
- A real back-reference-emitting compressor is possible but rarely worth building; if you do, the only
  acceptance test is a round trip — decompress your own output and assert it equals the input.

The trade-offs, a literal-only encoder, and the round-trip discipline are [C3.5](05-writing-compressed-back.md).

---

### Key takeaways

- 16-byte header; `decompSize` at `+0x08`; detect by the `JDLZ` magic before walking chunks.
- LZ77 with **two** flag streams: `flags1` = literal/reference, `flags2` = near/far reference.
- Back-reference copies may overlap the cursor — that is how runs are stored.
- The decompressor is ✅ verified: it reproduces the exact header size on all four global bundles.
- No standard compressor exists; write back uncompressed where you can, literal-only JDLZ where you must,
  and always round-trip-verify.

**Next:** [Chapter 4 — Byte-Level Toolcraft](../C4-Byte-Level-Toolcraft/C4-Byte-Level-Toolcraft.md), which
turns the readers, walkers, and codecs of C1–C3 into a disciplined workshop.
