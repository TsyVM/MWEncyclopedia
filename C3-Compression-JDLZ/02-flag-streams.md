# C3.2 — The Two Flag Streams

> **The one-sentence version:** JDLZ carries two independent 1-bit flag streams — one deciding
> literal-vs-reference, the other deciding near-vs-far — each refilled a byte at a time and tracked with
> a `0x100` sentinel that tells the decoder exactly when eight bits are spent.

[← C3.1 — Header & detection](01-header-and-detection.md) · [Chapter 3 hub](C3-Compression-JDLZ.md) ·
[Next: C3.3 — Near vs far references →](03-backreferences-near-far.md)

---

## What it is

The clever heart of JDLZ is that it splits its control bits into **two** streams instead of one:

- **`flags1`** — for each step, `0` means "emit a literal byte," `1` means "perform a back-reference."
- **`flags2`** — consulted only when `flags1` said "reference": `1` means a **near** reference, `0` means
  a **far** reference ([C3.3](03-backreferences-near-far.md)).

Each stream is a little bit-queue refilled one byte at a time from the compressed data, lowest bit first.

## How the `0x100` sentinel works

The elegant trick is how the decoder knows when a flag byte is exhausted without a separate counter. When
a stream needs refilling, it loads the next byte **with bit 8 set**:

```python
flags1 = data[pos] | 0x100     # 8 real flag bits in bits 0..7, plus a sentinel in bit 8
```

Now the decoder consumes bits from the bottom (`flags1 & 1`) and shifts right (`flags1 >>= 1`) after each
use. After exactly eight shifts, all eight real bits are gone and the only thing left is the sentinel,
which has walked down to bit 0 — so the value is now `1`. The refill condition is therefore simply "is
the stream equal to 1?":

```python
if flags1 == 1:                # sentinel reached bottom → 8 bits consumed → refill
    flags1 = data[pos] | 0x100
    pos += 1
```

This is a self-clocking counter folded into the value itself: no separate "bits remaining" variable, no
modulo arithmetic, just a marker bit that signals empty when it arrives at the bottom. The same mechanism
runs `flags2` independently.

## Why split into two streams

A single-stream LZ77 would interleave every decision into one bit queue, mixing "literal or reference"
with "which reference kind" and, in some designs, with length/distance bits too. JDLZ keeps the two
*independent binary decisions* in two separate streams. The payoff is packing efficiency and decoder
simplicity:

- Each stream stays a clean run of homogeneous decisions, so each flag byte is fully used for one kind of
  choice.
- The decoder's inner loop is branch-light: test `flags1` bit → maybe test `flags2` bit → act. No
  variable-width bit extraction for the control decisions themselves; the width is baked into the
  two-bytes-per-reference payload ([C3.3](03-backreferences-near-far.md)).

> 🟡 *Reasoned:* the "packing efficiency" motivation is the natural reading of why two streams beat one;
> the *mechanism* — two sentinel-tracked streams with the exact refill schedule below — is ✅ verified by
> the fact that the decompressor reproduces the retail bundles byte-for-byte.

## The refill schedule is by *step*, not by literal

The subtlety that trips up anyone writing an encoder: the streams are refilled based on how many *bits*
each has handed out, and a bit is handed out on every **step** (literal or reference for `flags1`; every
reference for `flags2`). The decoder consults `flags2` only on reference steps, but the *refill* of each
stream is driven purely by its own consumption. Get this schedule wrong on the encoding side and the two
streams drift out of phase — the decoder starts reading `flags2` bits that the encoder meant as
`flags1`, and the output is garbage from that point. This is exactly why a literal-only encoder must
mirror the decoder loop rather than "just emit literals" ([C3.5](05-writing-compressed-back.md)).

## Reading the decoder skeleton

Stripped to just the flag machinery:

```python
flags1, flags2 = 1, 1                 # both start "empty" so the first use refills
while output_not_done:
    if flags1 == 1: flags1 = data[pos] | 0x100; pos += 1
    if flags2 == 1: flags2 = data[pos] | 0x100; pos += 1
    if flags1 & 1:                    # reference
        # consult flags2 for near/far, read 2 payload bytes  (C3.3)
        flags2 >>= 1
    else:                             # literal
        # copy 1 byte
        pass
    flags1 >>= 1
```

Two things to notice. Both streams are *pre-checked for refill every iteration* — even `flags2` on a
literal step — which keeps their clocks in lockstep with the encoder. And `flags2` is only shifted on a
reference step, because that is the only time one of its bits was consumed.

## Bending it — the rules that keep the streams in phase

- **Refill both streams on the same schedule the decoder uses.** If you write JDLZ, replicate the exact
  "check `== 1`, refill with `| 0x100`" logic for both streams; do not invent your own bit counter.
- **Only shift `flags2` on reference steps.** Shifting it on a literal step consumes a bit the encoder
  didn't emit and desyncs the stream.
- **Verify by round trip.** The two-stream phase relationship is unforgiving; the only proof your encoder
  got it right is that decompressing your output reproduces the input exactly.

---

**Continue:** [C3.3 — Near vs far back-references](03-backreferences-near-far.md) ·
[Chapter 3 hub](C3-Compression-JDLZ.md)
