# C20.4 — EA-XAS v0 & IMA-ADPCM

> **The one-sentence version:** EA-XAS v0 is EA-XA's close relative for engine and movie audio — same
> predict-from-history idea, different block layout — and IMA-ADPCM is the industry-standard 4-bit block ADPCM
> driven by a stepsize table; both output PCM16.

[← C20.3 — EA-XA v2 decoded](03-eaxa-decode.md) · [Chapter 20 hub](C20-Audio-Codecs.md) ·
[Next: C20.5 — EA-MP3 →](05-eamp3.md)

---

## EA-XAS v0

EA-XAS ("XA Stream") is the variant used for **engine audio** ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md))
and **VP6 movie audio** ([Chapter 23](../C23-Video-VP6/C23-Video-VP6.md)). It shares EA-XA's core — a 4-bit
predictive ADPCM that reconstructs each sample from previous samples and a per-block predictor/shift with
**persistent history** ([C20.3](03-eaxa-decode.md)) — but arranges the bits in a different block layout tuned
for streaming.

The consequences for a decoder:

- **The prediction math is the same family** — predict from `(h1, h2)` with coefficient pairs, correct with a
  shifted residual, clamp, advance history.
- **The block framing differs** — how many samples per block and how the predictor/shift and residuals are
  packed. A decoder must use the XAS framing, not EA-XA's 15-byte block.
- **Still mono per stream** — the persistent predictor is per channel ([C20.2](02-replacement-rules.md)).

So if you can decode EA-XA, EA-XAS is a re-framing exercise, not a new algorithm — the reason mastering the
EA-XA math ([C20.3](03-eaxa-decode.md)) unlocks engine and movie audio too.

> ✅ *Verified (archive):* EA-XAS v0 is the engine/movie-audio codec, a close relative of EA-XA sharing the
> predictive-history design; it is mono per stream.
> 🟡 *Reasoned:* the exact XAS block byte-layout differences are a re-framing of the shared predictive core;
> the family relationship and usage are verified.

## IMA-ADPCM

IMA-ADPCM (tag `0x0011`) is the **industry-standard** 4-bit ADPCM — not EA-specific — so it's the most
portable of the four and decodable by well-known reference code. Its predictor is a **stepsize table** rather
than coefficient pairs:

```python
IMA_STEP_TABLE = [7, 8, 9, 10, 11, 12, 13, 14, 16, 17, ...]      # 89 entries
IMA_INDEX_TABLE = [-1, -1, -1, -1, 2, 4, 6, 8, -1, -1, -1, -1, 2, 4, 6, 8]

def decode_ima_nibble(nib, state):
    step = IMA_STEP_TABLE[state.index]
    diff = step >> 3
    if nib & 1: diff += step >> 2
    if nib & 2: diff += step >> 1
    if nib & 4: diff += step
    if nib & 8: diff = -diff
    state.pred = max(-32768, min(32767, state.pred + diff))       # clamp
    state.index = max(0, min(88, state.index + IMA_INDEX_TABLE[nib]))
    return state.pred
```

Each 4-bit nibble adjusts a running prediction by a fraction of the current **step size**, and the step size
itself walks up or down a table based on the nibble — adapting the quantiser to the signal. Like EA-XA, IMA is
**block-interleaved** with per-block state, so a decoder tracks `(pred, index)` across the block.

## Same shape, different predictors

The three ADPCM codecs share a shape — **4-bit residual + adaptive predictor + persistent state** — and differ
in the predictor:

| Codec | Predictor | State carried |
|---|---|---|
| EA-XA v2 | coefficient pairs + shift ([C20.3](03-eaxa-decode.md)) | last two samples `(h1, h2)` |
| EA-XAS v0 | coefficient pairs + shift (re-framed) | last two samples |
| IMA-ADPCM | stepsize table + index | `(pred, index)` |

So once you internalise "4-bit ADPCM = residual scaled by an adaptive predictor, state threaded across blocks,"
all three are the same idea with different predictor machinery. That shared shape is what makes one mental
model cover the game's compressed audio.

## Editing implications

- **Match the codec's framing.** Encode EA-XAS with XAS blocks, IMA with IMA blocks — the prediction family is
  shared but the packing isn't.
- **Thread state.** All three carry per-block state; drop it and you get boundary artifacts
  ([C20.3](03-eaxa-decode.md)).
- **Mono for the EA codecs**; IMA can be block-interleaved stereo but the game uses it per the container's
  channel count.
- **IMA is the portable fallback** — its reference decoders are ubiquitous, useful for validating your pipeline.

---

### Key takeaways

- EA-XAS v0 (engine/movie audio) is EA-XA's relative — same predictive-history core, different block framing.
- IMA-ADPCM (`0x0011`) is the standard 4-bit ADPCM with a **stepsize-table** predictor and `(pred, index)`
  state.
- All three ADPCM codecs share one shape: 4-bit residual + adaptive predictor + persistent per-block state.
- They differ only in predictor machinery (coefficient pairs vs stepsize table) and block layout.
- Match each codec's framing, thread its state, keep EA codecs mono, and use IMA as the portable reference.

**Continue:** [C20.5 — EA-MP3](05-eamp3.md) · [Chapter 20 hub](C20-Audio-Codecs.md)
