# C1.12 — The Runtime View: How the Engine Walks the Same Tree

> **The one-sentence version:** the game loads a file with the same recursive walk your dumper uses,
> then dispatches each recognised leaf to a registered handler and, for much of the data, points live
> structures straight at the loaded bytes — which is why alignment matters and why your parser and the
> engine always agree.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.11 — Failure modes & forensics](11-failure-modes.md) ·
[Next chapter: C2 — Identifiers & Hashing →](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)

---

## What it is

Everything in this chapter has been about how *your* tools read a file. The engine reads it almost
identically. Understanding the runtime side closes the loop: it explains why the format has the
properties it does (alignment, the container bit, header-exclusive sizes) and it gives you a mental
model precise enough to reason about what the game will and won't accept.

The engine's file layer has three parts you can name:

1. **The loader** — a generic recursive chunk walk, the same algorithm as [C1.3](03-walking-the-tree.md).
2. **The chunk-handler registry** — a table mapping recognised leaf ids to the subsystem callback that
   consumes their payload.
3. **In-place residency** — for much of the data, the loaded bytes are not copied into new structures;
   live objects point directly at the bytes in the loaded buffer.

## The loader and the handler registry

At startup the engine populates a registry: a set of `(chunk id → handler)` associations, one per
subsystem that cares about a chunk type. When the loader walks a bundle it does exactly what your
recursive walker does — test bit 31, recurse or not, advance `8 + size` — and at each *leaf* it looks
the id up in the registry and hands the payload to the matching handler. A leaf whose id is not
registered is simply stepped over. ✅ The existence of this dispatch table and its "look up id, call
handler, skip unknown" behaviour are confirmed in the shipped executable; the set of ids it registers is
a subset of the [master chunk-id table](../Glossary/chunk-ids.md).

This is the runtime counterpart of the two rules you already know:

- The **container bit** decides *recurse or not* — a purely structural decision the loader makes with no
  subsystem knowledge.
- The **registry** decides *who gets this leaf* — the semantic decision, deferred to whichever subsystem
  registered for the id.

Separating the two is what lets one generic loader serve every asset type in the game, and it is why a
file can contain chunk types a given build doesn't understand without breaking: unknown ids just aren't
in the registry, so they're skipped, exactly as [C1.1](01-the-container-bit.md) described.

## In-place residency and why alignment is real

For a large fraction of the data — geometry the GPU will read, textures it will sample, tables the
subsystems will index — the engine does **not** deserialise the bytes into fresh heap objects. It loads
the buffer once and points live structures directly into it. A vertex buffer *is* the bytes at
`MeshVertices`'s payload offset; the GPU is handed that region more or less directly.

This single fact explains several things this chapter treated as rules:

- **Why `0x11` alignment padding exists** ([C1.4](04-alignment-and-padding.md)): if a structure is going
  to be read in place by the GPU or by SIMD code, it must *start* on an aligned address. The padding
  guarantees that after the file is loaded contiguously, the real data lands on the boundary the
  hardware needs.
- **Why sizes must be exact** ([C1.2](02-chunk-header-and-sizes.md)): the loader uses `size` to know how
  many bytes belong to each region it maps. A wrong size doesn't just desync a walk — it hands a
  subsystem a region of the wrong length.
- **Why the world is kept Z-up with no conversion** ([C1.6](06-matrices-and-coordinates.md)): converting
  axes on load would mean rewriting every transform in the buffer, defeating the point of loading in
  place. The data is used in the coordinate system it was authored in.

The refcounted resource manager that owns these loaded buffers, hands out in-memory views of them, and
decides when they can be freed is the **StreamMgr**; it and the bundle/residency scheme are the subject
of [Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md) and [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md).
For this chapter, the point is narrower: because data is resident in place, the on-disk layout *is* the
in-memory layout, so your byte-level understanding of a file is also an understanding of the live object.

## Why your parser and the engine always agree

The equivalence is worth stating plainly because it is the foundation of all reverse engineering in this
book: **your correct walker and the engine's loader run the same algorithm over the same bytes.**
Consequences:

- If the game loads a file, a correct walker can parse it. There is no "secret" structure the game sees
  that your walk cannot, because the game's walk is your walk.
- If your walk desyncs on a file, the engine's would too — so a desync means either your walker is buggy
  (fix it, [C1.11](11-failure-modes.md)) or the file is genuinely corrupt (the game won't load it
  either).
- When you edit a file and your round-trip is byte-identical and your sizes balance, you have strong
  evidence the game will accept it — because "balanced size tree of registered chunks" is exactly what
  the loader requires.

That is why the no-op round-trip of [C1.11](11-failure-modes.md) is such a powerful test: it checks your
understanding against the *same* invariants the engine enforces.

## Bending it — reasoning about what the game will accept

- **Registered ids only, in a balanced tree.** If you synthesise a chunk, give it an id the target
  subsystem's handler recognises, set the container bit correctly, and let the size tree balance. A
  well-formed file of unregistered ids will *load* (they're skipped) but do nothing.
- **Respect in-place expectations.** Because data is used in place, don't hand the engine a structure on
  a misaligned boundary or of the wrong length. The padding and size rules aren't bureaucracy — they're
  what makes the in-place load correct.
- **Don't rely on the loader to "fix" anything.** It doesn't validate semantics; it walks, dispatches,
  and maps. Garbage in a well-formed chunk is garbage the subsystem will faithfully use. Correctness is
  your job; the loader only guarantees it can *reach* your bytes.

With the container model fully in hand — on disk and at runtime — you're ready for the next foundation:
the names inside these chunks are almost never text. [Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)
is about the hashes that stand in for them.

---

**Back to:** [Chapter 1 hub](C1-EAGL-Container-Model.md) ·
**Next chapter:** [C2 — Identifiers & Hashing](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)
