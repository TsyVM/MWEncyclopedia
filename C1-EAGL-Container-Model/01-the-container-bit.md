# C1.1 — The Container Bit (`0x80000000`)

> **The one-sentence version:** bit 31 of a chunk's id is a flag that says "my payload is *more
> chunks*, not data" — and that single bit is what makes every file in the game self-describing.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [Master chunk-id table](../Glossary/chunk-ids.md) ·
[Next: C1.2 — Chunk header & sizes →](02-chunk-header-and-sizes.md)

---

## What it is

Every chunk id is a 32-bit number. The top bit (`id & 0x80000000`) is not part of the "type" in any
semantic sense — it is a **structural flag**:

```c
static inline int is_container(uint32_t id) { return (id & 0x80000000u) != 0; }
static inline uint32_t chunk_type(uint32_t id) { return id & 0x7FFFFFFFu; }  // type without the flag
```

- **Bit set** → *container*. The payload is itself a sequence of `[id|size|payload]` chunks. You recurse.
- **Bit clear** → *leaf*. The payload is raw bytes in some format-specific layout. You stop and parse.

The lower 31 bits (`id & 0x7FFFFFFF`) are the actual type. That is why related chunks come in pairs
that differ only by the top bit: `0x80134000` is the **GeometryContainer**; `0x00134011` is a **leaf**
that lives inside it. Same family (`0x_0134___`), different role. If you strip the top bit from any
container id you get a number in the same numeric neighbourhood as the leaves it holds — the family
digits are shared, and only the container flag changes.

A useful way to hold it in your head: the id encodes two orthogonal things at once. The **family** (the
low bits) says *which subsystem* — geometry, textures, world, audio. The **container bit** (the top
bit) says *which shape* — a branch node or a leaf node. A dump tool reads the bit to decide the shape
and looks the family up in a table to name the subsystem.

## How the engine uses it

EAGL's loader is generic. It walks a file without knowing what the file *is*: at each chunk it tests
bit 31 and either descends into the payload as a fresh chunk stream, or hands the payload to whatever
subsystem registered for that id. Nothing in the walker is format-specific — the tree shape is carried
entirely by the ids themselves. This is the same trick that lets **your** dump tool
([C1.3](03-walking-the-tree.md)) print the structure of a file nobody has ever parsed before: you are
running the engine's own navigation logic.

Crucially, the walker's *arithmetic* does not depend on the bit. It advances `8 + size` either way. The
bit only changes whether the walker also *descends into* the payload it just skipped over. That
decoupling — "always advance by size; separately decide whether to recurse" — is why a flipped
container bit rarely crashes a walk outright; it just changes whether one subtree is read as structure
or as data. (More on that failure mode below, and in [C1.11](11-failure-modes.md).)

## Why it is designed this way

Two payoffs, both about **forward compatibility**:

1. **Safe skipping of the unknown.** A loader that doesn't recognise a container id can still step over
   it correctly using `8 + size`, *or* recurse into it and skip each unknown leaf inside. A tool built
   for one game version doesn't choke on a chunk added in a later one. The format degrades gracefully
   instead of corrupting.
2. **No schema needed to navigate.** Because "is this structure or data?" is answered by the data
   itself, EAGL never needs a separate table of "which ids are containers." The file is its own map. EA
   shipped this container model across many games on the engine precisely because new subsystems could
   add chunk types without touching the core loader.

> 🟡 *Reasoned, not byte-proven:* the "graceful degradation" intent is inferred from how the format
> behaves, not from a comment in the binary. But it is consistent across every EAGL title and is the
> only explanation that fits a generic, schema-free walker.

There is a second-order benefit that matters when you reverse-engineer: because the bit is structural
and universal, it is the *one* thing you can trust about a file you have never seen. You may not know
what `0x8003B810` means, but you know for certain it contains chunks, and you know exactly where it
ends. That single guaranteed fact is enough to map an entire unknown bundle.

## Bending it — the right way and the wrong way

**The right way.** The container bit is the safest thing you can lean on when reverse-engineering an
unknown file. Trust it absolutely: never guess whether to recurse, always read bit 31. When you build
a new tool, mirror the convention — if you author a new container, *set* the bit; if you write a leaf,
*clear* it. Stay inside the model and every other EAGL tool can still read your file.

**The wrong way — and what actually breaks:**

- **Clearing the bit on a real container** makes the loader treat a chunk tree as opaque data. The
  subsystem expecting children gets a meaningless byte blob; best case the asset silently fails to
  appear, worst case a downstream parser reads garbage lengths and walks off into the file.
- **Setting the bit on a real leaf** makes the walker recurse into raw data. The first four bytes of
  that data get misread as a child id and the next four as a child size — which is almost always an
  absurd, huge number — so the walk either bails immediately (you lose the chunk) or, if the bytes
  happen to look plausible, marches a bogus cursor through the rest of the payload and desyncs
  everything inside that branch.
- **The subtle one:** because the size field still advances the cursor by `8 + size` regardless of the
  bit, a *flipped* container bit usually doesn't crash the top-level walk — it quietly changes whether
  one subtree is interpreted as structure or data. The file "still opens," it's just wrong inside one
  branch. That makes it one of the nastier corruption bugs to chase, because nothing announces the
  fault.

The practical rule for modders: **you almost never set or clear this bit deliberately.** You preserve
it. The only time it matters is when you're synthesising a chunk from scratch, and then the rule is
simply "set it iff you're putting chunks inside."

---

**Continue:** [C1.2 — The chunk header & the off-by-eight](02-chunk-header-and-sizes.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
