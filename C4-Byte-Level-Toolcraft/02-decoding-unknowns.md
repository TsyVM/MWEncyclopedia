# C4.2 — Reading an Unknown Structure

> **The one-sentence version:** turn a blob into a layout with a disciplined loop — bound it, hypothesise
> a stride, test whether the records line up, corroborate across many instances, and anchor in the code —
> so that a decode is something you *prove*, not something you guess.

[← C4.1 — The core library](01-core-library.md) · [Chapter 4 hub](C4-Byte-Level-Toolcraft.md) ·
[Next: C4.3 — Hex-diffing →](03-hex-diffing.md)

---

## The loop

You have a leaf chunk of unknown meaning. Here is the method that decodes it, in order.

**1 — Bound it exactly.** Dump the chunk tree and read off the blob's start offset and `size`
([C1.9](../C1-EAGL-Container-Model/09-universal-opener.md)). You now know precisely which bytes you are
explaining and how many there are. Everything downstream leans on this number, so get it from the tree,
not from a guess.

**2 — Read the bytes as several types at once.** Print the blob as `u32`, as `f32`, and as ASCII, in
parallel columns. Patterns leap out: a plausible **count** (small integer near the front), **floats near
1.0** (a matrix `w`, a scale, a normalised value), **hashes** (high-entropy 32-bit values —
[C2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)), **ASCII runs** (names, FourCCs),
**`0x11` padding** ([C1.4](../C1-EAGL-Container-Model/04-alignment-and-padding.md)). This multi-view read
is the single most useful habit; a value that is noise as an integer is often obviously a float.

**3 — Hypothesise a record and a stride.** Most leaves are an array of fixed-size records, sometimes with
a small header. Guess the header size `H` and the record stride `S`. The test is arithmetic: `(size − H)`
should be divisible by `S`, and `(size − H) / S` should equal a count you can find elsewhere — often a
number stored in the header, or the entry count in a sibling hash table.

```python
def stride_fits(size, header, stride):
    body = size - header
    return body >= 0 and body % stride == 0, (body // stride if stride else 0)
```

**4 — Verify record *N+1* starts where you predicted.** Decode record 0 with your hypothesised layout,
step `S`, and decode record 1. If the fields of record 1 are as sensible as record 0 (the same field is a
float in both, the same field is a hash in both), your stride is right. If record 1 is garbage, `S` is
wrong — adjust and repeat. This is the same "does the next one line up?" logic as the chunk walk itself
([C1.11](../C1-EAGL-Container-Model/11-failure-modes.md)), applied one level down.

**5 — Corroborate across instances.** A layout that decodes *one* file is a coincidence generator; a
layout that decodes *the same structure in fifty files* is a fact. Run your hypothesised parser across
every file that contains the structure and check that every record stays sensible. Outliers are gold: a
record that breaks your layout is either a variant you must account for or a bug in your hypothesis
([C4.6](06-batch-recon.md)).

**6 — Anchor in the code.** The strongest confirmation is the routine that *reads* the structure. Find it
in the executable ([C4.4](04-static-analysis.md)) and let the disassembly tell you the field offsets and
types directly — a `mov eax, [esi+0x10]` in the reader is a load-bearing statement that offset 0x10 is a
dword field. When data-derived stride and code-derived offsets agree, the decode is ✅ verified.

## A worked shape

Suppose a leaf is 1,536 bytes and its parent's header stores the number 24. Test strides:

- `1536 / 24 = 64` exactly. Hypothesis: 24 records of 64 bytes, no header.
- Decode record 0 as `{u32 hash; f32 x,y,z; f32 m[12]; …}` — do floats 1–3 look like a position (world-
  scale numbers) and does float 15's neighbourhood hold a ~1.0? If so, 64-byte transformed instances is a
  strong hypothesis (and indeed the scenery instance stride;
  [C16](../C16-Scenery-Cull/C16-Scenery-Cull.md)).
- Step 64 bytes, decode record 1, confirm the same fields are the same types. Then run it across every
  scenery section in the world and confirm all records stay sane.

That is the whole method: an integer that divides, a next-record that lines up, a pattern that holds
across instances, and code that agrees.

## Why this beats guessing

Guessing a layout and "seeing if it looks right" fails silently — a wrong stride can still produce
plausible-looking floats for a few records before drifting. The loop defends against that at every step:
the divisibility test rejects impossible strides arithmetically; the next-record test catches drift
immediately; the cross-instance test catches layouts that fit one file by luck; and the code anchor
removes ambiguity entirely. Each stage narrows the hypothesis space until only the truth survives, and —
crucially — each stage tells you *how confident to be*, which is what feeds the confidence markers.

## Bending it — habits that make decoding fast

- **Always read multi-type.** `u32` + `f32` + ASCII side by side. Most "mystery" fields are obvious in the
  right view.
- **Let counts find strides.** A count in a header or a sibling table is the fastest way to pin `S`:
  `size / count` is your first stride to test.
- **Treat outliers as teachers.** The file that breaks your layout is showing you a variant or a bug;
  chase it rather than excluding it.
- **Stop at the code when data is ambiguous.** Two strides both "fit"? The reader routine breaks the tie
  ([C4.4](04-static-analysis.md)).

---

**Continue:** [C4.3 — Hex-diffing: change one thing, watch the bytes move](03-hex-diffing.md) ·
[Chapter 4 hub](C4-Byte-Level-Toolcraft.md)
