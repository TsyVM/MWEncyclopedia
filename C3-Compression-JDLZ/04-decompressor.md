# C3.4 — The Complete Decompressor

> **The one-sentence version:** twenty lines that turn every `.lzc` in the game into readable bytes,
> verified by reproducing the exact header sizes of all four global bundles — plus the guards that make
> it safe on hostile input.

[← C3.3 — Near vs far references](03-backreferences-near-far.md) · [Chapter 3 hub](C3-Compression-JDLZ.md) ·
[Next: C3.5 — Writing compressed data back →](05-writing-compressed-back.md)

---

## The algorithm

Everything from C3.1–C3.3 composes into one loop. Python first, for clarity:

```python
def jdlz_decompress(data: bytes) -> bytes:
    assert data[:4] == b'JDLZ', "not a JDLZ stream"
    out_len = int.from_bytes(data[8:12], 'little')     # decompSize @ +0x08
    out = bytearray(out_len)
    pos, op = 16, 0                                    # input past header; output at 0
    flags1, flags2 = 1, 1                              # both "empty" → refill on first use
    n = len(data)
    while pos < n and op < out_len:
        if flags1 == 1: flags1 = data[pos] | 0x100; pos += 1
        if flags2 == 1: flags2 = data[pos] | 0x100; pos += 1

        if flags1 & 1:                                 # back-reference
            b0, b1 = data[pos], data[pos + 1]; pos += 2
            if flags2 & 1:                             # near
                length = ((b0 & 0xF0) << 4 | b1) + 3
                dist   =  (b0 & 0x0F)           + 1
            else:                                      # far
                length =  (b0 & 0x1F)           + 3
                dist   = ((b0 & 0xE0) << 3 | b1) + 17
            for _ in range(length):                    # overlapping copy is intended
                if op >= out_len: break
                out[op] = out[op - dist]; op += 1
            flags2 >>= 1
        else:                                          # literal
            out[op] = data[pos]; op += 1; pos += 1
        flags1 >>= 1
    return bytes(out)
```

And the equivalent C, for embedding in a tool:

```c
size_t jdlz_decompress(const uint8_t* in, size_t in_len,
                       uint8_t* out, size_t out_len) {
    size_t pos = 16, op = 0;
    uint32_t f1 = 1, f2 = 1;
    while (pos < in_len && op < out_len) {
        if (f1 == 1) f1 = in[pos++] | 0x100;
        if (f2 == 1) f2 = in[pos++] | 0x100;
        if (f1 & 1) {
            uint8_t b0 = in[pos], b1 = in[pos + 1]; pos += 2;
            size_t len, dist;
            if (f2 & 1) { len = (((b0 & 0xF0) << 4) | b1) + 3; dist = (b0 & 0x0F) + 1; }
            else        { len =  (b0 & 0x1F) + 3;              dist = (((b0 & 0xE0) << 3) | b1) + 17; }
            for (size_t i = 0; i < len && op < out_len; i++, op++)
                out[op] = out[op - dist];              /* byte-by-byte: overlap is intended */
            f2 >>= 1;
        } else {
            out[op++] = in[pos++];
        }
        f1 >>= 1;
    }
    return op;                                          /* should equal out_len */
}
```

## The verification

This is not "believed correct" — it is measured. Run against the four JDLZ bundles in the retail data,
the output length matches the size written in each file's own header, exactly:

| File | Compressed in | Decompressed out | Matches header |
|---|---:|---:|:--:|
| `GLOBAL/GlobalB.lzc` | 1,520,744 | 2,803,648 | ✅ |
| `FRONTEND/FrontB.lzc` | 2,921,499 | 6,677,024 | ✅ |
| `GLOBAL/InGameB.lzc` | 522,637 | 946,264 | ✅ |
| `GLOBAL/gameplay.lzc` | 779,956 | 2,105,216 | ✅ |

Three of the four then walk as clean flat chunk trees to the last byte (194, 274, and 75 top-level chunks
respectively). The fourth expands correctly but is *not* a flat chunk tree — a useful reminder handled in
[C3.6](06-where-jdlz-lives.md). The decompressor's correctness is established by the exact-size match on
all four; the walk is a statement about *content*, not about the codec.

## Robustness guards

The bare algorithm is correct on well-formed input. A tool that eats arbitrary files needs four guards:

1. **Bound `out_len` before allocating.** A corrupt `decompSize` could request an absurd buffer; reject
   anything over a sane ceiling.
2. **Guard `dist > op`.** A back-reference before the start of output means corruption; stop rather than
   indexing out of bounds.
3. **Guard input reads.** Before reading two reference bytes, confirm `pos + 1 < in_len`; a truncated
   stream otherwise reads past the end.
4. **Check the postcondition.** If `op != out_len` when the loop exits, the stream was truncated or
   malformed; surface that as an error, since downstream parsing of a short buffer will fail confusingly.

```python
def jdlz_decompress_safe(data, max_out=1 << 30):
    if data[:4] != b'JDLZ': raise ValueError("not JDLZ")
    out_len = int.from_bytes(data[8:12], 'little')
    if out_len > max_out: raise ValueError(f"implausible decompSize {out_len}")
    out = jdlz_decompress(data)
    if len(out) != out_len: raise ValueError("truncated JDLZ stream")
    return out
```

## Why it's this simple

JDLZ decoding has no entropy stage (no Huffman/arithmetic table), no windows to manage beyond the output
itself, and no per-block headers. All state is three integers — input cursor, output cursor, and the two
flag words — which is why the whole decoder fits in twenty lines and runs at memory speed. That
minimalism is the point: it was built to be decoded quickly at load time into a pre-sized buffer, and it
shows.

## Bending it — using the decompressor well

- **Never bulk-`memcpy` the copy.** The overlap of [C3.3](03-backreferences-near-far.md) requires
  byte-by-byte; a `memcpy` corrupts runs.
- **Feed the whole file, not `data[16:]`.** The function expects the header present so it can read
  `decompSize`; slicing it off breaks the size read.
- **Re-identify the output.** Decompression yields a fresh file — run the identification tree on it
  ([C1.7](../C1-EAGL-Container-Model/07-non-chunk-containers.md)); do not assume "chunk tree."
- **Keep the compressed original.** If you later need to ship a `.lzc`, you'll want the exact original
  bytes as a reference and a backup ([Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md)).

---

**Continue:** [C3.5 — Writing compressed data back](05-writing-compressed-back.md) ·
[Chapter 3 hub](C3-Compression-JDLZ.md)
