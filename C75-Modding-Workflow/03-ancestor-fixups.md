# C75.3 — Ancestor-Size Fixups

> **The one-sentence version:** in a nested chunk container every chunk's size field includes its descendants, so
> when you change a chunk's size by Δ, *every ancestor up to the root* must have its size adjusted by the same Δ —
> and the freed or added bytes must be re-aligned, often by absorbing Δ into null padding to keep offsets on their
> 16/128-byte boundaries.

[← C75.2 — In-place vs. repack](02-inplace-vs-repack.md) · [Chapter 75 hub](C75-Modding-Workflow.md) ·
[Next: C75.4 — Verify by round-trip, then test →](04-verify-test.md)

---

## The size-tree

EAGL files are **nested** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)): a container chunk
holds child chunks, which hold their own children. Crucially, each chunk's **size field covers its entire payload —
*including* its nested children** ([C1.2](../C1-EAGL-Container-Model/02-chunk-header-and-sizes.md)). So the sizes form
a **tree**: the root's size includes its children's sizes, which include *their* children's, down to the leaves.

```
Root            size = 1000  ── includes everything below
├─ Container A  size = 600   ── includes B and C
│  ├─ Chunk B   size = 200
│  └─ Chunk C   size = 380
└─ Container D  size = 380
```

This tree is what the walker ([C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md)) uses to traverse the file:
it reads a chunk's size to know where the *next* chunk starts. So the sizes aren't decoration — they're the
*navigation*, and they must all be *consistent* or the walk desyncs.

## The fixup: adjust every ancestor

Now change chunk C's size by **Δ** (say you added 16 bytes: Δ = +16). Chunk C's own size field updates to 396 — but
that's not enough. Because C is *inside* Container A, **A's size must also grow by Δ** (600 → 616). And because A is
inside the Root, **the Root's size must grow by Δ too** (1000 → 1016). The rule:

> **When a chunk's size changes by Δ, every ancestor chunk up the tree to the root must adjust its size by Δ.**

```
change Chunk C: +16
  → Chunk C.size    += 16   (the edit itself)
  → Container A.size += 16   (C is inside A)
  → Root.size       += 16   (A is inside Root)
```

Walk from the changed chunk *up* to the root, adding Δ to each ancestor's size field. This is the **ancestor-size
fixup**, and it's the single most error-prone step in repacking ([C75.2](02-inplace-vs-repack.md)) — because *missing
one ancestor* leaves the tree inconsistent.

## Why missing one corrupts

If you update C and A but forget the Root, the file is now *internally contradictory*: the Root says it's 1000 bytes,
but its children actually total 1016. When the walker ([C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md))
traverses using the Root's stale size, it stops 16 bytes early — or reads 16 bytes of the next chunk as if they were
still inside the Root. Either way it **desynchronises**: every chunk after the error is read at the wrong offset,
producing garbage, wrong-type chunks, or a crash ([C1.11](../C1-EAGL-Container-Model/11-failure-modes.md)).

This is *the* classic EAGL corruption ([C75.2](02-inplace-vs-repack.md)): not a wrong *value* but a wrong *size*, so
the file's *structure* is broken even though its *content* looks fine. It's insidious because the edited chunk itself
is correct — the damage is in an *ancestor* you didn't think to touch. The fixup rule (adjust *every* ancestor)
exists precisely to prevent it: repackers must walk the full path to the root, never just the immediate parent.

## Alignment: absorb Δ into padding

Fixing the sizes isn't quite enough — there's also **alignment** ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)).
Record data is 16-byte aligned and sections are 2048-aligned ([C15.7](../C15-Track-Streaming/07-section-contents.md)),
so if Δ is *not* a multiple of the relevant alignment, growing a chunk by Δ pushes every *following* chunk off its
boundary — the misalignment hazard ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)) that corrupts
records even with correct sizes.

The technique that solves both at once is to **absorb Δ into null padding**:

- **Round the edit to alignment** — pad the changed chunk (or a neighbouring null chunk) so the *net* size delta is a
  multiple of 16 (or 128 for texture bases), keeping every following chunk on its boundary.
- **Re-synthesise null chunks** — insert or resize `0x00000000` null/padding chunks so downstream chunks keep their
  original start offset *mod* the alignment. A shrinking edit grows the tail padding; a growing edit is padded to the
  next boundary.
- **Recompute the local pad** — the `0x11` record-alignment padding ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md))
  of any chunk that moved must be recomputed for its new position, not copied.

So a correct repack does *three* things together: fix every ancestor's size (this page), keep everything aligned
(absorb Δ into padding), and recompute local pads. Do all three and the rebuilt file is structurally identical to a
clean one; miss any and it's subtly broken. This is why size-neutral edits ([C75.2](02-inplace-vs-repack.md)) are so
prized — they make Δ = 0, so *none* of this is needed.

> 🟡 *Reasoned:* the null-padding absorption technique (round Δ to alignment, re-synthesise null chunks to preserve
> mod-16/128 offsets, recompute local pads) is the method that reconciles size fixups with the verified alignment
> invariant ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)); it's how byte-for-byte rebuilds are
> achieved. The size-tree and the alignment invariant are verified ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md),
> [C63.6](../C63-Collision-World/06-ondisk-collision-data.md)).

## RE implications

- **The size-tree** — each chunk's size includes its descendants; the sizes are the walker's navigation
  ([C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md)).
- **The fixup** — change a chunk by Δ, and adjust *every ancestor* to the root by Δ; miss one and the walk desyncs.
- **The classic corruption** — a wrong *size* (structure) is worse than a wrong *value* (content); the damage is in
  an ancestor.
- **Alignment** — absorb Δ into null padding to keep offsets on 16/128 boundaries; recompute local pads.

---

### Key takeaways

- Each chunk's **size field includes its descendants**, so sizes form a **tree** that the walker uses to navigate
  ([C1.2](../C1-EAGL-Container-Model/02-chunk-header-and-sizes.md)–[C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md)).
- **The ancestor-size fixup** — when a chunk's size changes by Δ, **every ancestor up to the root** must adjust by Δ;
  walk from the edit *up* to the root, fixing each parent.
- **Missing one ancestor** leaves the tree **internally contradictory** — the walker desyncs, and every chunk after
  the error is read at the wrong offset (the **classic EAGL corruption**, [C1.11](../C1-EAGL-Container-Model/11-failure-modes.md))
  — a wrong *size*, not a wrong *value*.
- **Alignment must be preserved too** — if Δ isn't a multiple of 16/128, **absorb it into null padding** (re-synthesise
  null chunks, recompute local `0x11` pads) so following chunks stay on their boundaries
  ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)).
- A correct repack does **all three** — fix every ancestor size, keep alignment, recompute local pads — which is
  exactly why **size-neutral edits (Δ = 0)** are prized: they need *none* of it.

**Continue:** [C75.4 — Verify by round-trip, then test](04-verify-test.md) · [Chapter 75 hub](C75-Modding-Workflow.md)
