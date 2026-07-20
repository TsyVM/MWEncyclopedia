# C4.3 — Hex-Diffing: Change One Thing, Watch the Bytes Move

> **The one-sentence version:** the fastest way to find where a value lives is to let the game write it —
> change exactly one thing, save, and diff the bytes before and after; the bytes that moved are your
> field.

[← C4.2 — Decoding unknowns](02-decoding-unknowns.md) · [Chapter 4 hub](C4-Byte-Level-Toolcraft.md) ·
[Next: C4.4 — Static analysis →](04-static-analysis.md)

---

## What it is

Hex-diffing is controlled experiment applied to bytes. You take a baseline copy of a file, change *one*
value through some legitimate channel (a menu slider, a paint choice, a save-game action, an in-place
edit), capture the result, and compute the byte-level difference. Because you changed only one thing, the
diff is small and points straight at the field that encodes that thing. Instead of reasoning about where a
value *should* be, you make the system reveal where it *is*.

## The procedure

1. **Snapshot the baseline.** Copy the file. This is your `before`.
2. **Change exactly one variable.** One slider, one colour, one option. Discipline here is everything: two
   changes give you two diffs you can't tell apart.
3. **Capture `after`.** Copy the file again.
4. **Diff at the byte level** and report every run of changed bytes with its offset.

```python
def hexdiff(a_path, b_path):
    a = open(a_path, 'rb').read(); b = open(b_path, 'rb').read()
    n = max(len(a), len(b)); runs = []; i = 0
    while i < n:
        av = a[i] if i < len(a) else None
        bv = b[i] if i < len(b) else None
        if av != bv:
            start = i
            while i < n and (a[i] if i < len(a) else None) != (b[i] if i < len(b) else None):
                i += 1
            runs.append((start, a[start:i], b[start:i]))
        else:
            i += 1
    for off, old, new in runs:
        print(f"@0x{off:06X}  {old.hex(' ')}  ->  {new.hex(' ')}")
    return runs
```

5. **Interpret the run.** A four-byte run that changed from `00 00 80 3F` to `00 00 00 40` is a `float`
   going `1.0 → 2.0`. A single byte flipping `00 → 01` is a boolean or an enum. Read the changed run as
   `u32`/`f32`/`u8` and the type usually announces itself.

## Reading the diff like a scientist

The magic is in *controlling the variable*. If you nudge a tuning slider by one notch and exactly one
four-byte run changes, you have found that tuning field with certainty — no hypothesis required. A few
refinements make it even sharper:

- **Vary monotonically.** Change the value in known steps (1.0, then 2.0, then 3.0) across several
  captures. The field should change in a matching, monotonic way. If it doesn't, you found a re-encoded or
  derived value, not the raw one.
- **Watch for satellite changes.** Sometimes one logical change moves *two* runs: the value itself and a
  checksum, a count, or a size field. That second run is teaching you about the file's integrity
  machinery ([the ancestor-size tree](../C1-EAGL-Container-Model/02-chunk-header-and-sizes.md), for
  instance).
- **Beware the moving file.** If a change alters a length, everything after it shifts and the diff becomes
  huge. When that happens, prefer a change that keeps length constant, or align the two files by content
  before diffing.

## Where it shines, and where it doesn't

Hex-diffing is unbeatable for values the game itself writes back to disk: tuning that persists, paint and
customisation, save-game state, options. For these, it turns "where is the top-speed tuning field?" into a
two-minute experiment. It is the black-box counterpart to reading the code, and often faster for locating
a field (though the code tells you *why* and *what else uses it* — [C4.4](04-static-analysis.md)).

It does *not* help with values the game only reads (static asset data you can't change through a legitimate
channel) or values that never touch disk (pure runtime state). For those you decode structurally
([C4.2](02-decoding-unknowns.md)) or read the code. Knowing which technique a question calls for is part
of the craft: if the game will write the value for you, diff; if not, decode or disassemble.

## Why it's so reliable

The technique inherits its rigor from the experimental method: one controlled variable, a measured
response, repeated to confirm. There is essentially no room for a wrong conclusion when only one thing
changed and only one run moved — the causal link is direct. That is why hex-diffing sits at the top of the
black-box toolbox: it produces ✅-grade facts (this offset encodes this value) with almost no inference.

## Bending it — get clean diffs

- **One variable at a time, always.** The instant you change two things, you lose the attribution that
  makes the technique work.
- **Prefer length-preserving changes** so the diff stays small and local; a shifted file drowns the signal.
- **Confirm with a second step.** One capture locates a candidate; a monotonic second and third capture
  confirm it and reveal the encoding.
- **Log offset + before/after + meaning.** A diff you interpreted but didn't record is a fact you'll
  re-derive next week. Fold confirmed fields into your struct reference and your `mw` parser.

---

**Continue:** [C4.4 — Static analysis: the executable as ground truth](04-static-analysis.md) ·
[Chapter 4 hub](C4-Byte-Level-Toolcraft.md)
