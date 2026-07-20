# C20.3 — EA-XA v2 Decoded

> **The one-sentence version:** EA-XA reconstructs each 16-bit sample from a 4-bit residual, a per-block shift
> and predictor index, and the two previous samples via fixed coefficient pairs — so its history is
> *persistent across blocks*, and a 15-byte block carries 28 samples (a `0xEE`-headed 61-byte block is raw).

[← C20.2 — The two replacement rules](02-replacement-rules.md) · [Chapter 20 hub](C20-Audio-Codecs.md) ·
[Next: C20.4 — EA-XAS v0 & IMA-ADPCM →](04-xas-ima.md)

---

## The block

EA-XA v2 encodes audio in **28-sample blocks**. The common compressed block is **15 bytes**:

```
byte 0     : predictor index (high nibble) + shift (low nibble)
bytes 1–14 : 14 bytes × 2 nibbles = 28 four-bit residuals (one per sample)
```

An **uncompressed** block is 61 bytes, flagged by a `0xEE` header byte, and stores the 28 samples as raw PCM16
(with the trailing bytes carrying the history to seed the next block). Most blocks are the compressed 15-byte
form; the uncompressed form appears where the signal defeats prediction.

## The reconstruction

Each sample is **predicted** from the previous two samples and **corrected** by the block's residual:

```python
EA_XA_COEF = [(0, 0), (240, 0), (460, -208), (392, -220)]   # coefficient pairs, /256

def decode_eaxa_block(block, hist):
    h1, h2 = hist                                  # persistent history: last two samples
    if block[0] == 0xEE:                           # uncompressed 61-byte block
        samples = list(struct.unpack("<28h", block[1:57]))
        return samples, (samples[-1], samples[-2])
    pred, shift = block[0] >> 4, block[0] & 0x0F
    c1, c2 = EA_XA_COEF[pred]
    out = []
    for byte in block[1:15]:
        for nib in (byte & 0x0F, byte >> 4):       # two samples per byte
            s = nib - 16 if nib >= 8 else nib      # sign-extend the 4-bit residual
            sample = (s << shift) + ((h1*c1 + h2*c2) >> 8)   # predict + correct
            sample = max(-32768, min(32767, sample))          # clamp to PCM16
            out.append(sample)
            h2 = h1; h1 = sample                    # advance history
    return out, (h1, h2)
```

Three moving parts do the work:

- **The residual** (4-bit, sign-extended) is the small per-sample correction, scaled by **shift** — a per-block
  gain that adapts to signal amplitude.
- **The predictor** (`c1`, `c2`) forms the estimate from the last two samples; the block picks one of four
  coefficient pairs to match local signal character.
- **The clamp** keeps the result in PCM16 range.

## Persistent history is the defining property

The single most important thing about EA-XA: **history carries across blocks.** Block *k+1*'s first sample is
predicted from block *k*'s last two samples, so you cannot decode a block in isolation — you must thread
`(h1, h2)` from one block into the next:

```python
def decode_eaxa(payload):
    pcm, hist = [], (0, 0)                          # history starts at silence
    for block in split_blocks(payload):            # 15- or 61-byte blocks
        samples, hist = decode_eaxa_block(block, hist)
        pcm += samples
    return pcm
```

Forget to thread the history and each block starts from silence — you get a sound that's recognisable but
*clicks* at every 28-sample boundary, the classic EA-XA decode bug. The persistent predictor is exactly what
lets ~4 bits/sample reconstruct cleanly: you store only the residual, because the estimate comes free from
history.

> ✅ *Verified:* the coefficient pairs `[(0,0),(240,0),(460,−208),(392,−220)]`, the per-block predictor/shift
> byte, the 15-byte/28-sample block, the `0xEE` uncompressed form, and the persistent history are validated
> against real bank payloads.

## Encoding (for replacement)

To *write* EA-XA you invert the decode, and the hard part is the per-block choice. For each 28-sample block the
encoder must pick the **predictor pair and shift** that minimise reconstruction error, which practical encoders
do by **brute force** — try all four predictors and a range of shifts, decode each candidate, keep the lowest
error ([C19.6](../C19-Audio-Banks/06-editing-banks.md)). Because history is persistent, the encoder must also
carry `(h1, h2)` forward so its predictions match what the decoder will compute. This is why EA-XA encoders are
search-based rather than closed-form.

## Editing implications

- **Thread history on both decode and encode** — the persistent predictor is the whole codec; drop it and you
  get per-block clicks.
- **Mono only** — one predictor history per channel ([C20.2](02-replacement-rules.md)); de-interleave stereo.
- **Brute-force the predictor** when encoding — try all four pairs + shifts per block, keep the best.
- **Verify by decode-and-compare** — decode your re-encoded audio and check it against the source PCM
  ([C20.6](06-portable-decoder.md)).

---

### Key takeaways

- EA-XA is 4-bit predictive ADPCM in 28-sample blocks: a 15-byte compressed block (predictor/shift + 28
  residuals) or a `0xEE` 61-byte raw block.
- Each sample = `(residual << shift) + (h1·c1 + h2·c2) >> 8`, clamped, with coefficient pairs
  `[(0,0),(240,0),(460,−208),(392,−220)]`.
- **History is persistent across blocks** — thread `(h1, h2)` or every 28-sample boundary clicks.
- Encoding brute-forces the best predictor/shift per block and carries history forward.
- Decode and encode mono, thread history, and verify by decode-and-compare.

**Continue:** [C20.4 — EA-XAS v0 & IMA-ADPCM](04-xas-ima.md) · [Chapter 20 hub](C20-Audio-Codecs.md)
