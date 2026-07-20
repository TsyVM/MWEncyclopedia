# Chapter 20 — Audio Codecs

> **Goal of this chapter:** decode the handful of codecs every audio container in the game sits on — EA-XA v2,
> EA-XAS v0, IMA-ADPCM, and EA-MP3 — so that mastering the codec layer once unlocks banks, music, engine sound,
> and movie audio alike.

Chapter 19 found the sounds; this chapter turns their bytes into samples. The elegant thing about Most Wanted's
audio is that the *containers* are many (SNR, ABK, GIN, MUS, VP6 movie audio) but the *codecs* are few — a
small shared set. Learn EA-XA once and you can decode a car engine, a menu click, and the soundtrack, because
they all route through the same decoders.

> **Verified against retail data.** The codec set and its constraints are confirmed across the shipped audio;
> the EA-XA decode math (coefficient pairs, per-block predictor/shift, persistent history) is validated against
> real bank payloads. The landmark rate `44,100 Hz = 0xAC44` appears in the (big-endian) audio headers of
> Chapter 19.

---

## Deep-dive pages

- [C20.1 — The codec set](01-codec-set.md): the four codecs, their tags, and which containers use each.
- [C20.2 — The two replacement rules](02-replacement-rules.md): fixed-rate fields and mono-only ADPCM — the
  constraints every audio edit obeys.
- [C20.3 — EA-XA v2 decoded](03-eaxa-decode.md): the 4-bit predictive ADPCM — coefficients, per-block predictor,
  and persistent history.
- [C20.4 — EA-XAS v0 & IMA-ADPCM](04-xas-ima.md): the engine/movie variant and the standard block ADPCM.
- [C20.5 — EA-MP3](05-eamp3.md): MP3 wrapped in an EA header.
- [C20.6 — A portable decoder](06-portable-decoder.md): one decoder that dispatches on codec tag and handles
  every container.

---

## 20.1 Few codecs, many containers

The whole audio system rests on four codecs, identified by a tag in each sound's metadata
([C19.4](../C19-Audio-Banks/04-pt-records.md)):

| Tag (examples) | Codec | Where |
|---|---|---|
| `0x0011` | IMA-ADPCM | 4-bit block-interleaved, standard |
| `0xEA01` / `0x0A` | EA-XA v2 | sound banks & music (mono per stream) |
| `xas0` | EA-XAS v0 | engine audio & VP6 movie audio (mono per stream) |
| `0xEA03` / `0xEA` | EA-MP3 | MP3 in an EA header |

Because the codec set is shared, a container is "just a wrapper around a codec you already decode"
([C20.1](01-codec-set.md)). The bank/music/movie chapters differ in *routing*; the audio bytes are one of these
four.

## 20.2 Two rules govern replacement

Editing audio ([C19.6](../C19-Audio-Banks/06-editing-banks.md)) obeys two codec-level constraints:

- **Fixed rate fields** — ABK/GIN/MUS ignore a patched sample rate, so **resample to the original** (only SNR
  lets you patch the rate). Skip this and the sound plays at the wrong pitch/speed.
- **Mono-only codecs** — EA-XA/EA-XAS encode one channel at a time, so **de-interleave stereo to mono** before
  encoding, or the channels tangle into noise.

These two rules ([C20.2](02-replacement-rules.md)) are the difference between a clean audio swap and static.

## 20.3 EA-XA: predictive 4-bit ADPCM

EA-XA v2 is the workhorse. It's a **predictive** ADPCM: each sample is reconstructed from a 4-bit residual, a
per-block **shift** and **predictor index**, and the **two previous samples**. A 15-byte block encodes 28
samples (a predictor/shift byte + 14 nibble-pair bytes); a `0xEE`-headed 61-byte block is uncompressed. The
defining property is **persistent history**: block *k+1*'s first sample depends on block *k*'s last two, so the
decoder carries `(h1, h2)` across blocks ([C20.3](03-eaxa-decode.md)):

```
sample = (residual << shift) + ((h1·c1 + h2·c2) >> 8)   # predict + correct, then clamp
```

with the coefficient pairs `[(0,0),(240,0),(460,−208),(392,−220)]` (÷256). This persistent-history design is
why EA-XA is ~4 bits/sample yet reconstructs cleanly — and why the encoder must brute-force the best predictor
per block.

## 20.4 EA-XAS, IMA, and EA-MP3

The other three are variations on the theme ([C20.4](04-xas-ima.md), [C20.5](05-eamp3.md)):

- **EA-XAS v0** — a close relative of EA-XA used for engine and movie audio; same predictive idea, different
  block layout.
- **IMA-ADPCM** — the standard 4-bit block-interleaved ADPCM, a stepsize-table predictor.
- **EA-MP3** — ordinary MP3 frames wrapped in an EA header; decode the header, hand the frames to any MP3
  decoder.

All four produce PCM16 samples; they differ in how compactly and how they predict.

---

### Key takeaways

- Four shared codecs underlie every container: IMA-ADPCM (`0x0011`), EA-XA v2 (`0x0A`), EA-XAS v0 (`xas0`),
  EA-MP3 (`0xEA`).
- A container is a wrapper around one of these — decode the codec once, reuse everywhere.
- Two replacement rules: resample to the **fixed rate** (SNR excepted) and **de-interleave to mono** for ADPCM.
- EA-XA is predictive 4-bit ADPCM with **persistent history** across 28-sample blocks and per-block
  predictor/shift.
- EA-XAS, IMA, and EA-MP3 are variations; all output PCM16.

**Next:** [Chapter 21 — Music (MUS/MPF) & the Routing Graph](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md): the
interactive soundtrack built on these codecs.
