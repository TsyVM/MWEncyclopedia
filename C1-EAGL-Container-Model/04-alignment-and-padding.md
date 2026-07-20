# C1.4 — Alignment Padding (`0x11`) & Null Chunks

> **The one-sentence version:** runs of the byte `0x11` and zero-id "null chunks" are filler the engine
> inserts to align real data to memory boundaries — they carry no meaning, but if you add or remove
> them without fixing the surrounding sizes you'll desync the file.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.3 — Walking the tree](03-walking-the-tree.md) ·
[Next: C1.5 — Endianness islands →](05-endianness-islands.md)

---

## What it is

Two distinct kinds of filler appear in the chunk stream:

1. **`0x11` padding** — a run of the literal byte `0x11` at the *start of a payload*, before the
   meaningful fields, used to push the real data to an alignment boundary.
2. **Null chunks** — a full chunk whose `id == 0x00000000`, sitting *between* chunks as a spacer. It has
   a normal header and you step over it like any other chunk (`8 + size`), but its payload is
   meaningless.

Stripping the first is trivial:

```python
def strip_pad(p):
    i = 0
    while i < len(p) and p[i] == 0x11:
        i += 1
    return p[i:]
```

If your struct fields come out shifted by a few bytes, an unstripped `0x11` run is the usual culprit —
it is the single most common "everything is offset by 3" bug when you first parse a new leaf.

## How alignment works here

The engine loads many of these files by streaming them into memory and pointing structures directly at
the bytes (especially geometry and textures, which the GPU reads). For that to work, certain structures
must begin on an aligned address — commonly a 16-byte boundary. Rather than store an explicit "padding
length," EAGL pads with a recognisable, non-zero sentinel (`0x11`) so a reader can just *skip while it
sees `0x11`* and land on the first real byte. The null chunk does the same job at the chunk
granularity: it occupies space to align the *next* chunk's header.

The two operate at different granularities, and that is the whole reason both exist:

- `0x11` aligns **data within a payload** — the leaf's fields start on a boundary.
- A null chunk aligns **the start of the next chunk header** — the next sibling starts on a boundary.

> ✅ *Verified (world-data chunks):* record-bearing world chunks align their data by the exact rule
> **`alignedStart = (ptr + 0x17) & ~0xF`** — 16-byte alignment of the first record past the 8-byte chunk header
> (`0x17` = `8` + `0xF` round-up bias; `& ~0xF` clears the low four bits) — and stream **sections are 2048-aligned**,
> so section-relative alignment equals absolute alignment
> ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md), [C15.7](../C15-Track-Streaming/07-section-contents.md)).
> The `0x11` filler is precisely the bytes this rule skips.
> 🟡 *Varies by family:* the granularity differs elsewhere — TPK texture *pixel* bases additionally use 128-byte
> alignment — so confirm the specific boundary for the chunk family you're editing. The 16-byte record rule and the
> 2048-byte section stride are verified; a repack must preserve both ([C75.3](../C75-Modding-Workflow/03-ancestor-fixups.md)).

## Why `0x11` specifically

Two practical reasons a non-zero sentinel beats zero-fill:

- **It's distinguishable from a real null.** Zero bytes are everywhere in binary data — counts of zero,
  empty fields, terminators. A `0x11` run is an unambiguous "skip me" signal that won't collide with
  legitimate zero-valued data. If the padding were zeros, a reader could not tell "aligning filler"
  from "a field that happens to be 0."
- **It's cheap to detect.** "Skip while byte == 0x11" needs no length field and no lookahead. The
  reader self-synchronises on the first non-`0x11` byte, with no state.

Zero-id null chunks coexist with `0x11` because they solve the *chunk-boundary* problem, where a
sentinel-byte scan wouldn't help — the walker is reading headers, not scanning bytes, so the filler has
to be a well-formed (if meaningless) chunk it can step over with the normal `8 + size`.

## Bending it — padding is the friendliest thing to touch, with one catch

**The right way (this is genuinely safe):**

- **Re-padding after an edit.** When you rebuild a chunk and need it to land on a boundary, you can add
  or trim `0x11` bytes freely — padding has no semantic content. This is how repackers keep alignment
  intact after changing payload lengths.
- **Stripping before parsing.** Always strip leading `0x11` before mapping a struct. It costs nothing
  and removes the most common class of alignment bug.

**The catch — and the wrong way:**

- Padding bytes still **count toward `size`.** A `0x11` run at the front of a payload is part of that
  payload's length; a null chunk's `8 + size` is part of its parent's length. So if you *remove* 5
  bytes of padding, you've shortened a payload by 5 — and every ancestor size is now off by 5 (back to
  the [ancestor-size rule](02-chunk-header-and-sizes.md)). Padding is meaningless, but it is not *free*:
  it occupies counted bytes.
- **Removing alignment the engine actually needs** can break in-place loaders that expect a structure
  at an aligned address. If a texture or vertex buffer suddenly starts on an odd boundary, the GPU
  upload or a SIMD read can misbehave. When in doubt, *preserve* alignment rather than minimise it — the
  few wasted bytes are never worth the risk.

Rule of thumb: treat padding as adjustable but *counted*. Add or remove it to hit a boundary, then fix
the size tree. Never assume it can simply be deleted to "clean up" a file.

---

**Continue:** [C1.5 — Endianness islands](05-endianness-islands.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
