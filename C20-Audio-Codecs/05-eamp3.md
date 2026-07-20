# C20.5 — EA-MP3

> **The one-sentence version:** EA-MP3 is ordinary MP3 frames wrapped in an EA header — strip the header, hand
> the frames to any MP3 decoder — so it's the one game codec you don't have to implement yourself.

[← C20.4 — EA-XAS v0 & IMA-ADPCM](04-xas-ima.md) · [Chapter 20 hub](C20-Audio-Codecs.md) ·
[Next: C20.6 — A portable decoder →](06-portable-decoder.md)

---

## MP3 in an EA wrapper

EA-MP3 (tags `0xEA`, `0xEA03`) is exactly what it sounds like: **standard MPEG-1 Layer III (MP3) frames**
inside an EA container header. Unlike the ADPCM codecs, there is no EA-specific decode math to reverse — the
payload, once unwrapped, is MP3 that any decoder (`libmpg123`, `minimp3`, the platform decoder) plays. EA's
contribution is the *framing*: how the MP3 stream is chunked and described within the audio container
([Chapter 19](../C19-Audio-Banks/C19-Audio-Banks.md), [21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)).

## Decoding

The decode is unwrap-then-delegate:

```python
def decode_eamp3(payload):
    mp3_frames = strip_ea_header(payload)      # remove the EA framing → raw MP3 frames
    return mp3_decode(mp3_frames)              # any standard MP3 decoder → PCM
```

Two practical points:

- **Find the frame sync.** MP3 frames begin with a sync word (`0xFFE`…); the EA header precedes the first
  frame, and the frames themselves are self-describing (bitrate, rate, channels in each frame header).
- **Stereo is native.** MP3 handles stereo internally, so — unlike the mono-only EA ADPCM codecs
  ([C20.2](02-replacement-rules.md)) — EA-MP3 sounds can be genuine stereo in one stream.

## Why EA-MP3 exists alongside ADPCM

Given the game already has three ADPCM codecs, why also carry MP3? Because they occupy different points on the
size/quality/CPU curve:

- **ADPCM (EA-XA/XAS/IMA)** — cheap to decode, ~4 bits/sample, good for the *many* short sounds (effects,
  engine grains) where decode cost and count matter.
- **EA-MP3** — better quality per byte at higher decode cost, good for *long* audio (music, some streams) where
  the quality-per-megabyte win outweighs the CPU.

So a title picks MP3 where a track is long and quality-sensitive, and ADPCM where sounds are numerous and
short. The codec tag on each sound records which trade was made.

> ✅ *Verified (archive):* EA-MP3 is MP3 wrapped in an EA header; decoding is unwrap-then-delegate to a standard
> MP3 decoder.
> 🟡 *Reasoned:* the exact EA header framing bytes are the container's detail; the "standard MP3 inside"
> identity is verified.

## Editing implications

- **Re-encode as MP3.** A replacement for an EA-MP3 sound is an MP3 stream re-wrapped in the EA framing — encode
  to MP3, then apply the EA header.
- **Stereo is allowed.** Rule 2 (mono-only, [C20.2](02-replacement-rules.md)) does **not** apply to EA-MP3 —
  keep stereo if the original was stereo.
- **Rate rule still applies.** For fixed-rate containers, match the original rate ([C20.2](02-replacement-rules.md)).
- **Match the framing.** Keep the EA header format and any block/section structure the container expects
  ([C21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)).

---

### Key takeaways

- EA-MP3 (`0xEA`/`0xEA03`) is standard MP3 frames in an EA header — no EA-specific decode math.
- Decode = strip the EA header → hand frames to any MP3 decoder → PCM.
- MP3 is natively stereo, so the mono-only ADPCM rule does **not** apply to EA-MP3.
- It coexists with ADPCM because it wins on quality-per-byte for long audio, at higher decode cost.
- Replace by re-encoding to MP3 and re-wrapping in the EA framing; keep stereo and match the rate/framing.

**Continue:** [C20.6 — A portable decoder](06-portable-decoder.md) · [Chapter 20 hub](C20-Audio-Codecs.md)
