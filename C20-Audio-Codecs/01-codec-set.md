# C20.1 — The Codec Set

> **The one-sentence version:** four codecs cover all of the game's audio — IMA-ADPCM, EA-XA v2, EA-XAS v0, and
> EA-MP3 — each identified by a tag in the sound's metadata, so a container is just a wrapper around a codec you
> already decode.

[← Chapter 20 hub](C20-Audio-Codecs.md) · [Next: C20.2 — The two replacement rules →](02-replacement-rules.md)

---

## The four codecs

Every sound in the game — bank effect, engine loop, music section, movie audio — is encoded with one of four
codecs, named by a tag in the sound's metadata ([C19.4](../C19-Audio-Banks/04-pt-records.md)):

| Tag(s) | Codec | Bits/sample | Channels | Used by |
|---|---|---|---|---|
| `0x0011` | **IMA-ADPCM** | ~4 | block-interleaved | standard ADPCM sounds |
| `0xEA01`, `0x0A` | **EA-XA v2** | ~4 | mono per stream | sound banks, music |
| `xas0` | **EA-XAS v0** | ~4 | mono per stream | engine audio, VP6 movie audio |
| `0xEA03`, `0xEA` | **EA-MP3** | variable | stereo (MP3) | some music/streams |

Three are 4-bit ADPCM variants (compact, predictive); one is MP3 (EA-wrapped). All decode to PCM16.

## A container is a wrapper around a codec

The organising insight of the whole audio system: **containers route and describe; codecs decode.** SNR, ABK,
GIN, MUS, and VP6 movie audio are different *containers* ([Chapter 19](../C19-Audio-Banks/C19-Audio-Banks.md),
[21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md), [22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md),
[23](../C23-Video-VP6/C23-Video-VP6.md)), but the audio bytes inside each are one of these four codecs. So:

```
container (SNR / ABK / GIN / MUS / VP6)
   → metadata: codec tag + rate + channels + payload location
   → codec (IMA / EA-XA / EA-XAS / EA-MP3)
   → PCM16 samples
```

Master the codecs once and every container's audio opens. This is why the book separates the two: the bank
chapters teach *finding and describing* a sound; this chapter teaches *decoding* it.

## Why EA reused so few codecs

The small shared set is a deliberate engineering economy:

- **One decoder set, many uses.** The runtime ships one implementation of each codec; every container routes
  through them. Less code, fewer bugs, easier hardware acceleration on the platforms of the era.
- **ADPCM for footprint.** EA-XA/EA-XAS at ~4 bits/sample is a huge saving over PCM16 (16 bits/sample) across
  the game's hundreds of sounds — roughly a 4× reduction — which matters on a disc-based console/PC title.
- **MP3 where quality wins.** EA-MP3 covers cases where MP3's quality-per-byte beats ADPCM (longer music), at
  the cost of more decode work.

So the codec choice per sound trades size against quality and CPU, but always within this fixed menu.

## Reading the codec tag

The first thing a decoder does is read the tag and dispatch:

```python
def decode_sound(codec_tag, payload, rate, channels):
    if codec_tag in (0x0A, 0xEA01):  return decode_eaxa(payload)      # C20.3
    if codec_tag == "xas0":          return decode_eaxas(payload)     # C20.4
    if codec_tag == 0x0011:          return decode_ima_adpcm(payload) # C20.4
    if codec_tag in (0xEA, 0xEA03):  return decode_eamp3(payload)     # C20.5
    raise ValueError(f"unknown codec tag {codec_tag!r}")
```

Because the set is closed and small, the `raise` at the bottom is a real tripwire: hitting it on retail data
means you misread the tag (wrong byte order — the tag lives in a big-endian header
[C19.4](../C19-Audio-Banks/04-pt-records.md)), not that there's a fifth codec.

## Editing implications

- **Keep the codec** on replacement — encode your audio with the *same* codec the sound used, so the container's
  decoder still handles it ([C19.6](../C19-Audio-Banks/06-editing-banks.md)).
- **Read the tag in the right byte order** — it's in a big-endian audio header for `PT`/`SC*` sounds.
- **Reuse one decoder set** — build the four decoders once and point every container at them.

---

### Key takeaways

- Four codecs cover all game audio: **IMA-ADPCM**, **EA-XA v2**, **EA-XAS v0**, **EA-MP3** — three ADPCM, one
  MP3, all → PCM16.
- Containers route and describe; codecs decode — the audio bytes inside any container are one of the four.
- The small shared set is an economy: one decoder set, ADPCM for footprint, MP3 where quality wins.
- Dispatch on the codec tag (read big-endian); a closed set means an unknown tag is a parse error.
- On edits, keep the sound's codec and reuse a single decoder set across containers.

**Continue:** [C20.2 — The two replacement rules](02-replacement-rules.md) · [Chapter 20 hub](C20-Audio-Codecs.md)
