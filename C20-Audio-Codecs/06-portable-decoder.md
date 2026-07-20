# C20.6 — A Portable Decoder

> **The one-sentence version:** one decoder that reads the codec tag and dispatches to the four codecs handles
> *every* audio container in the game — bank, music, engine, movie — because they all sit on the same codec
> layer.

[← C20.5 — EA-MP3](05-eamp3.md) · [Chapter 20 hub](C20-Audio-Codecs.md) ·
[Next: Chapter 21 — Music (MUS/MPF) →](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)

---

## One decoder, every container

Because containers route and codecs decode ([C20.1](01-codec-set.md)), a single codec-dispatch function serves
the whole game. Give it a codec tag, the payload, and the format, and it returns PCM16:

```python
def decode_audio(codec_tag, payload, rate, channels):
    if codec_tag in (0x0A, 0xEA01):   pcm = decode_eaxa(payload)        # C20.3
    elif codec_tag == "xas0":         pcm = decode_eaxas(payload)       # C20.4
    elif codec_tag == 0x0011:         pcm = decode_ima_adpcm(payload)   # C20.4
    elif codec_tag in (0xEA, 0xEA03): pcm = decode_eamp3(payload)       # C20.5
    else: raise ValueError(f"unknown codec {codec_tag!r}")
    return pcm, rate, channels

def extract_sound(container_entry, container_payload):
    meta = container_entry                       # from SNR/ABK-PT/GIN/MUS (Chapters 19,21,22)
    audio = container_payload[meta.offset : meta.offset + meta.size]
    return decode_audio(meta.codec, audio, meta.rate, meta.channels)
```

Every container's job is to produce that `meta` (codec, rate, channels, offset, size) — the SNR entry
([C19.2](../C19-Audio-Banks/02-snr-spt.md)), the ABK `PT` ([C19.4](../C19-Audio-Banks/04-pt-records.md)), the
MUS `SCHl` ([C21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)), the GIN grain
([C22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)). Once you have `meta`, decoding is container-agnostic.

## The full pipeline

Putting the two chapters together, extracting any sound is:

```
container (Ch 19/21/22/23)
  → parse metadata: codec tag + rate + channels + payload location  (right byte order!)
  → slice the payload
  → decode_audio() dispatches to the codec (this chapter)
  → PCM16 samples → WAV / analysis / mixer
```

The two failure modes to guard are both about *reading the metadata*, not the codec:

- **Wrong byte order** on the codec tag / rate (they're in big-endian `PT`/`SC*` headers —
  [C19.4](../C19-Audio-Banks/04-pt-records.md)).
- **Wrong offset/size** slicing the payload.

Get `meta` right and the codec layer just works.

## Validating the decoder

A portable decoder earns trust by round-tripping and cross-checking:

1. **Decode to WAV and listen** — the ultimate test.
2. **Cross-check duration** — decoded sample count ÷ rate should equal the metadata's `durationMs`
   ([C19.2](../C19-Audio-Banks/02-snr-spt.md)); a mismatch means a rate or block-framing error.
3. **Check for boundary clicks** — EA-XA/XAS clicks at block boundaries mean history isn't threaded
   ([C20.3](03-eaxa-decode.md)).
4. **Compare IMA against a reference** — IMA-ADPCM has ubiquitous reference decoders; matching one validates
   your block handling ([C20.4](04-xas-ima.md)).

These checks localise a bug to *metadata reading* (duration off), *history threading* (clicks), or *framing*
(reference mismatch) — the three things that go wrong.

## The payoff

The reason to build this one decoder is leverage: it turns "the game's audio is five different container
formats" into "the game's audio is one codec layer with five front-ends." Extract a menu click, a car engine, a
music section, and a cutscene's dialogue with the *same* decode call, differing only in which container produced
the `meta`. That is the whole argument of Chapters 19–20: **learn the codec layer once, and all of the game's
sound opens.**

## Editing round-trip

The decoder is also half of the *edit* round-trip ([C19.6](../C19-Audio-Banks/06-editing-banks.md)):

```
original sound → decode_audio() → PCM   (this decoder)
edit PCM → apply the two rules (C20.2) → encode → fit to span → write back to the container
verify: decode the result again, compare duration/level/character to the intent
```

Decoding validates every encode: if your re-encoded sound decodes back to the right duration, character, and no
clicks, the edit is sound. The decoder you built to *read* audio is the tool that *verifies* every audio edit.

---

### Key takeaways

- One `decode_audio(codec_tag, …)` dispatch handles all four codecs and therefore every container.
- Each container's only job is to produce `meta` (codec, rate, channels, offset, size); decoding is
  container-agnostic.
- The pipeline is parse metadata (right byte order) → slice → dispatch → PCM; the failure modes are metadata,
  not codec.
- Validate by listening, duration cross-check, boundary-click check, and IMA reference comparison.
- The same decoder verifies edits — decode the re-encoded sound and compare to intent.

**Continue:** [Chapter 21 — Music (MUS/MPF) & the Routing Graph](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md) ·
[Chapter 20 hub](C20-Audio-Codecs.md)
