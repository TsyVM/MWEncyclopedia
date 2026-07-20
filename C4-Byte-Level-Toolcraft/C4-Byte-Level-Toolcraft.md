# Chapter 4 — Byte-Level Toolcraft

> **Goal of this chapter:** turn the primitives of C1–C3 into a disciplined workshop. By the end you have
> a reusable library, a repeatable method for decoding an undocumented structure, a way to read the
> executable's own code as ground truth, and validation harnesses that catch your mistakes before the
> game does.

The first three chapters gave you the three primitives of the data: **structure** (the chunk tree),
**names** (the three hashes), and **packing** (JDLZ). This chapter is different in kind — it is about
*how you work*. Reverse engineering Most Wanted is not a matter of already knowing the formats; it is a
matter of having a method reliable enough that you can decode a structure nobody has documented and
*know* you got it right. That method is the most transferable thing in the book, and it is what separates
guessing from engineering.

Everything here is the method this encyclopedia itself was built with. The hash function of
[C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md) was not assumed — it was read out of the
instruction stream and proven three ways. The JDLZ codec of [C3.4](../C3-Compression-JDLZ/04-decompressor.md)
was not trusted — it was measured against four retail bundles. This chapter teaches you to do the same.

---

## Deep-dive pages

- [C4.1 — The reusable core library](01-core-library.md): bounded readers/writers, the tree walker, the hashes, and the codec, assembled into one small module you import everywhere.
- [C4.2 — Reading an unknown structure](02-decoding-unknowns.md): the disciplined loop — dump, hypothesise a struct, test the stride, corroborate across instances — for turning a blob into a layout.
- [C4.3 — Hex-diffing: change one thing, watch the bytes move](03-hex-diffing.md): the highest-yield technique for finding where a value lives, by making the game write it for you.
- [C4.4 — Static analysis: the executable as ground truth](04-static-analysis.md): mapping virtual addresses to file offsets, reading disassembly, and recognising the patterns (loops, structs, vtables) that pin a fact down.
- [C4.5 — Validation harnesses](05-validation.md): round-trip tests, cross-file corroboration, and confidence discipline — how you know, and how you say how well you know.
- [C4.6 — Batch reconnaissance](06-batch-recon.md): running your tools across the whole data set to find outliers, catalogue every file, and turn one decoded structure into a survey.

---

## 4.1 The workshop mindset

Three principles run through everything that follows.

**Ground every claim in evidence, and label the evidence.** A fact about the game is worth exactly as
much as the thing that backs it. A stated struct offset you can see in a hex editor is certain. A stride
that reproduces across a hundred instances is verified. An inference about *why* a field exists is
reasoned, not proven, and must be marked so. This is the ✅/🟡/⏳ discipline of the
[main README](../README.md#confidence-markers), and it is not decoration — it is what makes the reference
trustworthy, because a reader always knows how much weight a line bears.

**Prefer the source that can't lie.** There is a hierarchy of evidence. The executable's code is the
ground truth — it is literally what the game does. The shipped data is next — it is what the code
consumes. The behaviour of the running game is real but harder to attribute. Community lore is a
hypothesis to test, never a citation. When two sources disagree, the one closer to the machine wins, and
the disagreement itself is worth documenting ([C4.5](05-validation.md)).

**Make the tool prove itself.** Never trust a parser you haven't round-tripped or a fact you haven't
reproduced. The single most powerful habit in the book is: after you decode something, write the code
that *checks* you decoded it — the no-op round trip ([C1.11](../C1-EAGL-Container-Model/11-failure-modes.md)),
the reproduce-a-known-value test ([C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)), the
size-match measurement ([C3.4](../C3-Compression-JDLZ/04-decompressor.md)). If it passes across the whole
data set, you were right; if it fails on one file, that file is teaching you something.

## 4.2 The core library, at a glance

Rather than re-deriving readers in every experiment, you keep one small module — call it `mw` — that
exports the primitives from C1–C3: a bounded `Reader`, the recursive `walk_tree`, the three hashes, the
JDLZ codec, and the universal `open_eagl`. Every later chapter's code assumes it exists. Building it once,
correctly, is [C4.1](01-core-library.md); the point is that your experiments become *short* because the
plumbing is already solved and tested.

## 4.3 The decoding loop

When you meet an undocumented structure, you do not guess a layout and hope. You run a loop:

1. **Dump** the surrounding chunk tree so you know the blob's exact bounds and size
   ([C1.9](../C1-EAGL-Container-Model/09-universal-opener.md)).
2. **Hypothesise** a record layout — a field order and a stride — from the size and any obvious values
   (counts, floats near 1.0, hashes, ASCII).
3. **Test the stride:** does `blob_size / stride` come out an integer? Does record *N+1* start where the
   hypothesis says?
4. **Corroborate across instances:** the same structure in ten other files should decode sensibly with
   the same layout. A layout that only works on one file is wrong.
5. **Anchor in code where you can:** find the routine that reads the structure and let the disassembly
   confirm the field offsets ([C4.4](04-static-analysis.md)).

This is [C4.2](02-decoding-unknowns.md) in full, and it is how every struct in this book was pinned down.

## 4.4 Two techniques worth singling out

**Hex-diffing** is the highest-yield black-box technique: change one value in the game (a tuning slider, a
paint colour), save, and diff the file's bytes before and after. The bytes that moved are the value you
were hunting — the game located the field for you. [C4.3](03-hex-diffing.md).

**Static analysis** is the highest-yield white-box technique: the executable contains the exact code that
reads every structure, and reading that code resolves questions the data alone leaves ambiguous. You need
only a little scaffolding — a virtual-address-to-file-offset mapping and a disassembler — to turn
`speed.exe` into the authoritative spec it actually is. [C4.4](04-static-analysis.md).

---

### Key takeaways

- The method is the transferable skill: dump, hypothesise, test the stride, corroborate, anchor in code.
- Evidence has a hierarchy — code over data over behaviour over lore — and every claim carries its
  confidence marker.
- Build the core library once so experiments stay short; make every tool prove itself with a round-trip
  or a reproduced known value.
- Hex-diffing finds where a value lives by making the game write it; static analysis settles what the
  data can't by reading the code that consumes it.

**Next:** [Chapter 5 — Textures: the TPK Container Model](../C5-Textures-TPK/C5-Textures-TPK.md), the first
real format we take apart with these tools.
