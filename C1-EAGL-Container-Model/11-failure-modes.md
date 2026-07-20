# C1.11 — Failure Modes & Forensics

> **The one-sentence version:** when a walk desyncs, the corruption is almost always a single wrong
> size, a flipped container bit, an unstripped pad, or a wrong-endian field — and each leaves a distinct
> fingerprint you can read straight off the dump.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.10 — Editing & repacking](10-editing-and-repacking.md) ·
[Next: C1.12 — The runtime view →](12-runtime-view.md)

---

## What it is

A "desync" is when your walker's cursor stops landing on real chunk headers and starts reading payload
bytes as `{id, size}`. From that point the dump is nonsense: absurd ids, gigantic sizes, a walk that
bails early or explodes. The skill this page teaches is **reading the nonsense backwards to the byte
that caused it** — because in a size-tree format the failure is nearly always local and singular.

## The four fingerprints

Almost every chunk-file corruption is one of these, and each looks distinct in a dump:

**1. A wrong size (the desync).** The dump is correct up to some chunk, then the very next "chunk" has an
implausible id and size. The last *good* chunk is the culprit: its `size` sent the cursor to the wrong
place. If the bogus size is *slightly* off, you overshot or undershot a real boundary (a hand-edited
size, or a repack that forgot the ancestor fixup). Confirm by checking whether the last good chunk's
`8 + size` lands exactly on the next real header — it won't.

```
[C] 0x80134010 GeometryObject   size=4176   @0x94     ← last sane line
    0x6F42A013                  size=2036291584  @0x…  ← garbage: previous size was wrong
```

**2. A flipped container bit.** The walk *doesn't* desync — the top-level structure still looks fine —
but one subtree is wrong. Either a container is being read as a leaf (a branch that should have children
shows as a single opaque leaf) or a leaf is being read as a container (the dumper tries to recurse into
raw data and emits a burst of nonsense ids *inside* one chunk, then resyncs at the next sibling because
the size stride was still correct). The tell is that the damage is *contained to one branch* while
everything around it is fine — the signature of [C1.1](01-the-container-bit.md)'s subtle failure.

**3. An unstripped pad / wrong field offset.** The tree walks perfectly — all ids and sizes are sane —
but a *leaf's fields* come out shifted: a name reads with leading `0x11` bytes, a float's `w` canary
isn't ≈ 1.0, a count is implausibly large. This isn't a tree problem at all; it's a
[C1.4](04-alignment-and-padding.md) padding-strip you skipped inside `visit()`.

**4. A wrong-endian field.** The tree is fine and the *layout* is fine, but a value is byte-reversed: a
44.1 kHz rate reads as `0x44AC0000`, a channel count is in the thousands. You are inside an EA audio
island reading it little-endian ([C1.5](05-endianness-islands.md)). Swap and move on.

## The forensic toolkit

**The no-op round-trip.** Parse the file and re-serialise it *without editing anything*
([C1.10](10-editing-and-repacking.md)). If the output isn't byte-identical to the input, your parser or
serialiser is wrong — and the *first differing byte* points straight at the misunderstanding. This is
the most powerful single diagnostic in the chapter: it separates "my tool is wrong" from "the file is
wrong" definitively, before you blame the game.

```python
orig = open(path, 'rb').read()
tree = parse(orig)
rebuilt = serialise_to_bytes(tree)
i = next((k for k in range(min(len(orig), len(rebuilt))) if orig[k] != rebuilt[k]), None)
print("identical" if orig == rebuilt else f"first diff at 0x{i:X}")
```

**Bisect the edit.** If an edited file won't load but the original did, and a no-op round-trip *is*
identical (so your tooling is sound), the fault is in your edit. Halve it: apply only the first half of
your changes, test; then the other half. A binary search over a batch of edits finds the offending one
in a few loads.

**Compare dumps.** Dump the original and the edited file and diff the two trees. A correct edit changes
*only* the leaf you touched and the `size=` values of its ancestors (each by the same `delta`). If any
*other* id, size, or offset moved, your repack disturbed something it shouldn't have.

**Check the sums by hand at the break.** At the last-good chunk, verify `parent.size == Σ(8 +
child.size)` for that parent. The child whose inclusion breaks the sum is the one whose size is wrong —
or the parent's own size is the wrong one. Two lines of arithmetic localise it.

## Why these are the failure modes

The size-tree format has very few degrees of freedom, and each maps to one fingerprint: the *stride* can
be wrong (fingerprint 1), the *recurse decision* can be wrong (2), the *within-payload offset* can be
wrong (3), or the *byte order* can be wrong (4). There is almost nothing else to get wrong at this layer,
which is exactly why diagnosis is tractable: the format is simple enough that its failures are, too.
Higher-level corruption (a bad vertex, a wrong hash) lives in later chapters; at the container layer,
these four cover the field.

## Bending it — turn failures into guardrails

- **Make the round-trip test a unit test.** Run it across a whole directory of the game's files. A
  parser that round-trips every retail file is a parser you can trust to edit them.
- **Validate before you save.** Re-walk your serialised bytes and assert every size balances *before*
  writing to disk. It is far cheaper to fail in your tool than to chase a black screen in the game.
- **Keep the original.** Every forensic technique here compares against the original bytes. Lose them and
  you lose your ground truth — which is why [Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md)
  makes the backup the very first step.

---

**Continue:** [C1.12 — The runtime view: how the engine walks the same tree](12-runtime-view.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
