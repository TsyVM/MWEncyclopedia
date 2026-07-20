# C1.10 — The Size Tree in Practice: Editing & Repacking

> **The one-sentence version:** there are exactly two ways to edit a chunk file — change values without
> changing lengths (trivial and safe), or change lengths and then repair every ancestor size up to the
> root (powerful and error-prone) — and knowing which one you're doing is the whole game.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.9 — Universal opener](09-universal-opener.md) ·
[Next: C1.11 — Failure modes & forensics →](11-failure-modes.md)

---

## What it is

Editing a chunk file falls into two categories with very different risk profiles:

- **In-place edit** — you overwrite bytes *without changing any payload length*. No size field changes,
  the tree stays balanced automatically, and the edit is as safe as editing gets.
- **Repack** — you change the length of some payload (add vertices, swap in a larger texture, insert a
  chunk). Now the edited leaf's size changed, so every container that encloses it has the wrong size and
  must be corrected from the leaf up to the file root.

The first is the workhorse of practical modding; reach for it whenever you possibly can. The second is
unavoidable when data genuinely grows or shrinks, and it is where files get corrupted if the discipline
slips.

## In-place editing

The safe pattern: find the absolute offset of the field (your dumper's `@0x…` column plus the offset of
the field within the payload), and overwrite exactly those bytes.

```python
import struct

def patch_float(path, abs_offset, value):
    with open(path, 'r+b') as f:
        f.seek(abs_offset)
        f.write(struct.pack('<f', value))
```

Because no length changed, no size field anywhere in the file is now wrong. This is why "find the float,
overwrite the float" is the single most reliable technique in the game: a tuning value, a colour, a flag
— any fixed-width field — can be changed this way with zero risk to the structure. Whole categories of
modding (retuning handling in a vault, recolouring a light, flipping a boolean) never leave this mode.

The only rule: **keep the length identical.** A four-byte float stays four bytes; a fixed-width name
field stays its full width (pad with the format's convention, usually zeros or the field's own padding).
If you cannot keep the length, you are repacking, not patching.

## Repacking and the ancestor-size fixup

When a payload's length changes by `delta` bytes, every ancestor's `size` must change by the same
`delta`. Walk the parent chain from the edited leaf to the root and add `delta` to each:

```python
def rebuild_sizes(node):
    """Bottom-up: a container's size = sum over children of (8 + child.size).
       `node` is your in-memory tree with .is_container, .children, .payload, .size."""
    if node.is_container:
        total = 0
        for c in node.children:
            rebuild_sizes(c)
            total += 8 + c.size
        node.size = total
    else:
        node.size = len(node.payload)
    return node.size
```

The robust way to repack is therefore **not** to poke the file in place, but to:

1. Parse the file into an in-memory tree (containers with children, leaves with payloads).
2. Make your edits on the tree — change a leaf's payload, insert or remove a chunk.
3. Call `rebuild_sizes(root)` so every container's size is recomputed from its children.
4. Serialise the tree back out, writing each `{id, size}` header followed by its payload.

```python
def serialise(node, out):
    if node.is_container:
        body = bytearray()
        for c in node.children:
            serialise(c, body)
        out += struct.pack('<II', node.id, len(body)) + body
    else:
        out += struct.pack('<II', node.id, len(node.payload)) + bytes(node.payload)
```

Because `serialise` writes container sizes from the actual serialised children, it *cannot* produce an
unbalanced tree — the size is derived, never hand-typed. This is the key insight: **make size a computed
output, not a value you edit.** Every corruption bug in this space comes from someone editing a size by
hand and getting the arithmetic wrong.

## Alignment during a repack

Repacking interacts with the `0x11` padding and null chunks of [C1.4](04-alignment-and-padding.md). If a
structure needs to start on a 16-byte boundary, a repacker re-emits the right amount of `0x11` padding
(or a null chunk) so the boundary is preserved after lengths shift. Since the padding is part of the
payload it precedes (or a chunk in its own right), `rebuild_sizes` counts it automatically — you only
have to *emit* the right amount, not account for it separately.

## Atomic writes

Never write in place over the only copy. The safe sequence, expanded in
[Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md):

1. Write the new bytes to a temp file next to the target.
2. `fsync`/flush so the bytes are really on disk.
3. Rename the temp over the target (atomic on the same volume).
4. Keep the original as a `.bak` until you've confirmed the game loads the result.

A crash mid-write then costs you a temp file, not your game data.

## Why it is designed this way

The size-tree design trades a little editing friction for a lot of parsing simplicity. A format that
stored explicit child *offsets* would be easy to edit-in-place-with-growth (bump the offsets) but hard
to validate; the size-tree format is trivial to validate (do the sums balance?) at the cost of the
ancestor fixup. EA optimised for a fast, generic loader, and the modder inherits the fixup as the price.
The good news is that the fixup is entirely mechanical — `rebuild_sizes` does it perfectly every time,
so the price is paid once, in code.

## Bending it — the safe envelope and the cliff edge

- **Stay in-place whenever the data allows it.** Most tuning, colour, and flag edits never need a
  repack. Reach for repacking only when data truly changes length.
- **Never hand-edit a size field.** Recompute it. The instant you type a size by hand you have
  reintroduced the exact bug the derived-size approach eliminates.
- **Round-trip test before you touch content.** Parse → serialise → compare bytes. If a *no-op*
  round-trip doesn't reproduce the original file, your serialiser is wrong and you must fix it *before*
  trusting it with edits. (This test is powerful enough that [C1.11](11-failure-modes.md) uses it as a
  diagnostic.)
- **Preserve trailing slack and padding unless you understand them.** "Cleaning up" filler is how a
  file that loaded fine stops loading.

---

**Continue:** [C1.11 — Failure modes & forensics](11-failure-modes.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
