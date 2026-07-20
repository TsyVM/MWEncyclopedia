# C1.2 — The Chunk Header & the Off-by-Eight

> **The one-sentence version:** `size` counts payload bytes only, so every parent chunk's size is the
> sum of its children *including their 8-byte headers* — and almost every corruption bug in EAGL
> editing is a size field that no longer adds up.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.1 — The container bit](01-the-container-bit.md) ·
[Next: C1.3 — Walking the tree →](03-walking-the-tree.md)

---

## What it is

The header is eight bytes, little-endian:

```c
struct ChunkHeader {
    uint32_t id;    // type + container bit
    uint32_t size;  // PAYLOAD length, excluding these 8 header bytes
};
```

The defining subtlety is in the comment: `size` is the payload only. To reach the next sibling you step
`8 + size`, not `size`. Stepping `size` is the **off-by-eight** — you land 8 bytes early, read a
mid-payload byte as the next id, and the whole walk after that point is rubbish.

It is worth internalising the two cursor quantities and never confusing them:

- **`size`** — what the header stores: payload bytes.
- **`8 + size`** — the *stride*: how far the cursor moves to the next sibling.

Every bug in this section is ultimately a place where those two got swapped.

## How sizes nest

Because a container's payload *is* its children, a parent's `size` equals the total on-disk footprint
of everything inside it — and each child contributes `8 + child.size` (its own header plus its
payload). Sizes therefore form a strict accounting tree:

```
parent.size  ==  Σ over children ( 8 + child.size )
```

This recurses all the way down. Worked example: a `GeometryObject` container (`0x80134010`) holding a
160-byte object header leaf and a mesh container. The header leaf contributes `8 + 160 = 168`. If the
mesh container's own `size` is `4000`, it contributes `8 + 4000 = 4008`. So the object container's
`size` must be `168 + 4008 = 4176` — and *its* parent, the `GeometryContainer`, counts `8 + 4176`
toward its own size, and so on to the file root.

The consequence that bites every modder: **if you change the length of a leaf payload, every ancestor's
size is now wrong** and must be fixed from the leaf upward to the file root. This is the "ancestor-size
fixup" that [C1.10](10-editing-and-repacking.md) and [Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md)
treat as a hard rule.

## Why it is designed this way

Excluding the header from `size` makes the walker's arithmetic uniform: the cursor advance is *always*
`8 + size`, with no special case for "does this count its own header." It also means a chunk's payload
length is exactly the number you'd pass to a `read(payload, size)` call — the header is the walker's
concern, the payload is the subsystem's concern, and the two never overlap. Clean separation, at the
cost of one gotcha you only learn once.

The alternative — a size that *included* the header — would force every consumer to subtract 8 before
reading a payload and would make an empty chunk have `size == 8` instead of `size == 0`. EA chose the
convention that keeps the payload number honest and pushes the one addition into the walker, which does
it in exactly one place.

> ✅ *Verified:* the `8 + size` stride and header-exclusive sizing are confirmed against retail PC v1.3
> across every chunk-tree file in the game; a parser that steps `size` desyncs immediately on the first
> multi-chunk file.

## Bending it — what the size field lets you get away with, and what it doesn't

**What the engine tolerates (the "right way" to exploit it):**

- **Trailing slack after the last chunk.** A walker stops when fewer than 8 bytes remain, so bytes past
  the final chunk are ignored. This is occasionally useful as a scratch area, but don't rely on it
  surviving a repack — a rebuilder will drop it.
- **In-place edits that keep length constant.** If your edit doesn't change any payload length, *no*
  size needs touching. This is the entire basis of safe in-place patching — change values, never
  counts, and the size tree stays valid. It is the single most reliable modding technique in the game,
  and the reason so much practical modding is "find the float, overwrite the float."

**What breaks (the "wrong way"):**

- **A short size.** If a leaf's `size` is smaller than the data it really holds, the walker steps into
  the middle of that data and reads the tail as a phantom chunk header. Downstream desync.
- **A long size.** If `size` overruns the buffer (`off + 8 + size > len`), a correct walker bails at
  that chunk — you lose it and everything after. A *careless* walker reads out of bounds and crashes.
  The bounds check in [C1.3](03-walking-the-tree.md) exists precisely to turn a crash into a clean stop.
- **A parent whose size doesn't equal the sum of its children.** Two failure modes depending on sign:
  too-large and the parent "contains" bytes belonging to its next sibling (it eats the sibling); too
  small and the parent ends mid-child (the child gets truncated and the sibling walk resyncs onto
  garbage). Either way the branch is silently wrong while the file still "opens."

The takeaway is the discipline, not the dread: **edit payloads freely, but treat every size field as a
derived quantity.** Recompute sizes bottom-up after any length change, or keep lengths constant and
sidestep the problem entirely. The forensic side — *how to tell which* size broke when a file won't
load — is [C1.11](11-failure-modes.md).

---

**Continue:** [C1.3 — Walking the tree](03-walking-the-tree.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
