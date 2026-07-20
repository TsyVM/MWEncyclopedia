# C3.5 — Writing Compressed Data Back

> **The one-sentence version:** you rarely need a real JDLZ compressor — write the file back uncompressed
> when the loader will accept it, emit a literal-only JDLZ stream when the container demands compression,
> and never trust any encoder you haven't round-tripped.

[← C3.4 — The complete decompressor](04-decompressor.md) · [Chapter 3 hub](C3-Compression-JDLZ.md) ·
[Next: C3.6 — Where JDLZ lives →](06-where-jdlz-lives.md)

---

## The three options, in order of preference

There is no widely-used general-purpose JDLZ *compressor*, and for almost all modding you don't need one.
Your options, best first:

1. **Write it back uncompressed.** When you rebuild a bundle that arrived as `.lzc`, you can usually save
   it without the `JDLZ` header. The loader reads a raw chunk tree fine when the magic is absent; the file
   is larger on disk but loads correctly. This is the simplest and safest path and the right default.
2. **Emit a literal-only JDLZ stream.** When a container *requires* a JDLZ payload — the minimap stores
   each tile as a JDLZ chunk and the loader expects compression there ([C29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md))
   — you produce a valid JDLZ file that happens to use only literals. It is game-readable and correct,
   just larger than an optimal compressor would make it.
3. **Write a real back-reference-emitting compressor.** Possible, occasionally worth it for size, but the
   only acceptance test that matters is a round trip: decompress your own output and assert it equals the
   input. Getting the two-stream phase right ([C3.2](02-flag-streams.md)) is the hard part.

## When can you get away with uncompressed?

The rule of thumb: if the *content* is what the loader consumes (a chunk tree it walks), and compression
was merely a storage optimisation, uncompressed usually works. If the *container* around the content has
a slot that is defined as "a JDLZ blob" — where the loader itself calls the decompressor on that slot —
you must provide something the decompressor accepts, which means option 2. The safe move when unsure is
to try uncompressed first in a disposable copy and confirm the asset appears; if it doesn't, fall back to
literal-only.

Always keep the original compressed bytes as a backup before you replace them
([Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md)).

## A literal-only encoder

A literal-only stream never emits a back-reference: every `flags1` bit is 0. The one subtlety is that you
must still supply `flags2` refill bytes on the decoder's schedule, because the decoder pre-checks *both*
streams for refill every step ([C3.2](02-flag-streams.md)) — even though it never *consults* a `flags2`
bit when `flags1` says "literal." The robust way to get the interleave right is to **mirror the decoder
loop**: run the same `flags1`/`flags2` sentinel logic and emit a refill byte at the exact moment the
decoder would consume one.

```python
def jdlz_compress_literal(raw: bytes) -> bytes:
    out = bytearray()
    out += b'JDLZ'
    out += bytes([0x02, 0x10, 0x00, 0x00])            # version, headerSize=16, reserved
    out += len(raw).to_bytes(4, 'little')             # decompSize
    comp_size_pos = len(out)
    out += b'\x00\x00\x00\x00'                        # compSize (patched at end)

    # Mirror the decoder's flag clocks so the two streams stay in phase.
    f1 = f2 = 1
    i = 0
    while i < len(raw):
        if f1 == 1:                                   # decoder would refill flags1 here
            out.append(0x00);  f1 = 0x100             # a flags1 byte of all-literal (zero) bits
        if f2 == 1:                                   # decoder would refill flags2 here
            out.append(0x00);  f2 = 0x100             # a flags2 byte (contents irrelevant for literals)
        out.append(raw[i]); i += 1                    # the literal byte itself
        f1 >>= 1                                       # consumed one flags1 bit (literal step)
        # note: flags2 is NOT shifted on a literal step (see C3.2)
    out[comp_size_pos:comp_size_pos + 4] = len(out).to_bytes(4, 'little')
    return bytes(out)
```

> The exact refill interleave must match the decoder, and the safest way to be sure is to
> **round-trip-test**: `jdlz_decompress(jdlz_compress_literal(x)) == x` for a range of inputs, including
> ones whose length is not a multiple of 8. If a length-7 or length-9 buffer fails, your refill schedule
> is off by a step.

## The golden rule

> **Decompress → edit → write back uncompressed if the loader allows; only re-compress when the container
> demands it; and always verify a compressor by decompressing its own output.**

This keeps you in the safe, reversible envelope. Uncompressed output has no encoder bugs to hit;
literal-only output has exactly one thing that can go wrong (the refill phase) and the round-trip test
catches it; a real compressor multiplies the ways to be subtly wrong, which is why it is the last resort.

## Bending it — encoder discipline

- **Prefer size cost over correctness risk.** A bundle that is a few hundred KB larger but definitely
  loads beats a tightly-compressed one that might desync.
- **Round-trip every encoder, on ragged lengths.** The failure mode is phase drift, and it hides on
  multiple-of-8 inputs; test 7, 9, 15, 17-byte buffers explicitly.
- **Patch `compSize` last.** Write a placeholder, then overwrite it with the final length; a wrong
  `compSize` can cause a reader to reject or truncate the file.
- **Keep the original `.lzc`.** It is your reference for both correctness and rollback.

---

**Continue:** [C3.6 — Where JDLZ lives, and what it doesn't tell you](06-where-jdlz-lives.md) ·
[Chapter 3 hub](C3-Compression-JDLZ.md)
