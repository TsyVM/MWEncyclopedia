# C3.3 — Near vs Far Back-References

> **The one-sentence version:** a back-reference reads two payload bytes and unpacks them into a `length`
> and a `distance` two different ways — a short-distance "near" form and a long-distance "far" form — and
> the copy may overlap the write cursor, which is exactly how runs of repeated bytes are stored.

[← C3.2 — The two flag streams](02-flag-streams.md) · [Chapter 3 hub](C3-Compression-JDLZ.md) ·
[Next: C3.4 — The complete decompressor →](04-decompressor.md)

---

## What it is

When `flags1` selects a reference, the decoder reads two bytes `b0 b1` from the stream and consults one
`flags2` bit to choose the unpacking. Both forms yield a `length` (how many bytes to copy) and a
`distance` (how far back in the already-produced output to copy from):

```
flags2 bit == 1  →  NEAR
    length   = ((b0 & 0xF0) << 4 | b1) + 3
    distance =  (b0 & 0x0F)            + 1

flags2 bit == 0  →  FAR
    length   =  (b0 & 0x1F)            + 3
    distance = ((b0 & 0xE0) << 3 | b1) + 17
```

The copy takes `length` bytes from `distance` bytes before the current output position:

```python
for _ in range(length):
    out[op] = out[op - dist]
    op += 1
```

These formulas are exact and ✅ verified — they are exercised thousands of times, both paths, in the
byte-perfect decompression of the retail bundles.

## How the bit-packing divides the space

The two forms carve the (length, distance) plane differently, and the split is deliberate:

**Near** spends most of its bits on **length** and few on distance:

- `distance` is the low nibble of `b0` (`b0 & 0x0F`), plus 1 → range **1…16**. Short reach.
- `length` is the high nibble of `b0` promoted by 8 bits, OR'd with all of `b1` (`(b0 & 0xF0) << 4 | b1`),
  plus 3 → range **3…4098**. Long copies.

So *near* is "copy a lot, from just behind" — ideal for the common case of nearby repetition and, above
all, for **runs**: a run of identical bytes is `distance = 1`, `length = N`, and the overlapping copy
regenerates the whole run one byte at a time.

**Far** spends most of its bits on **distance** and few on length:

- `length` is the low five bits of `b0` (`b0 & 0x1F`), plus 3 → range **3…34**. Short copies.
- `distance` is the top three bits of `b0` promoted by 8 bits, OR'd with `b1` (`(b0 & 0xE0) << 3 | b1`),
  plus 17 → range **17…2064**. Long reach.

So *far* is "copy a little, from further back" — for matches that repeat something seen a while ago. The
`+17` offset on far distance is not arbitrary: near already covers distances 1…16, so far begins at 17
and the two forms tile the distance axis without wasteful overlap.

## Why overlapping copies matter

The line `out[op] = out[op - dist]` inside a forward loop is doing something that looks like a bug and is
actually the format's most important feature. When `dist < length`, the copy reads bytes it has only just
written. Concretely, `distance = 1, length = 5` starting from a single `A`:

```
before:  … A            (op points just past A, dist=1)
step 1:  … A A          (copied out[op-1] = A)
step 2:  … A A A
step 3:  … A A A A
step 4:  … A A A A A
```

One two-byte reference expands a single byte into a five-byte run. This is how LZ77 encodes
runs-of-identical-bytes and periodic patterns without a separate RLE mechanism, and it is why you must
implement the copy as a **byte-by-byte forward loop**, never as a bulk `memcpy` — a `memcpy` with
overlapping source and destination has undefined behaviour and would corrupt exactly the run case the
format relies on.

## Why two forms instead of one

A single reference encoding would have to pick one (length, distance) budget and live with it — either
short reach (bad for distant repeats) or short length (bad for runs). By offering two forms selected by a
single cheap `flags2` bit, JDLZ lets the encoder choose, per reference, whether this match is a
long-copy-from-nearby or a short-copy-from-afar, and pack the two bytes accordingly. It is a small,
elegant way to get two useful operating points out of the same two-byte payload.

> 🟡 *Reasoned:* the "two operating points" rationale is the natural design reading; the bit-exact
> formulas and the `+1/+3/+17` biases are ✅ verified by round-trip against retail data.

## Bending it — implementing the copy correctly

- **Always copy byte-by-byte, forward.** Overlap is intended; bulk-copy primitives will corrupt runs.
- **Guard `dist > op`.** A reference that reaches before the start of the output is a corrupt stream;
  detect and reject rather than reading `out[negative]`.
- **Respect the `+3` minimum length.** Both forms bias length by at least 3 because a match shorter than
  the two-byte reference cost would not be worth encoding — the encoder would emit literals instead.
  Knowing the minimum is 3 helps when you reason about or build an encoder.
- **Don't overrun `decompSize`.** Clamp the copy so the last reference cannot write past the pre-sized
  output buffer; a well-formed stream ends exactly at `decompSize`.

---

**Continue:** [C3.4 — The complete decompressor](04-decompressor.md) · [Chapter 3 hub](C3-Compression-JDLZ.md)
