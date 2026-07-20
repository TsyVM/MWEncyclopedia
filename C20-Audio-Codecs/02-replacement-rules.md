# C20.2 — The Two Replacement Rules

> **The one-sentence version:** every audio edit obeys two codec-level constraints — resample to the
> container's **fixed sample rate** (only SNR lets you patch it) and **de-interleave stereo to mono** for the
> mono-only ADPCM codecs — or the sound plays at the wrong pitch or dissolves into noise.

[← C20.1 — The codec set](01-codec-set.md) · [Chapter 20 hub](C20-Audio-Codecs.md) ·
[Next: C20.3 — EA-XA v2 decoded →](03-eaxa-decode.md)

---

## Rule 1 — fixed rate fields

Most of the game's audio containers **do not honour a patched sample-rate field** on replacement. ABK, GIN, and
MUS play the replacement at the sound's *original* rate regardless of what you write into the rate field. So if
your replacement audio is a different rate than the original, you must **resample it to the original rate
first** — otherwise it plays at the wrong pitch and speed (a 48 kHz clip dropped into a 44.1 kHz slot plays
~9 % fast and sharp).

**The one exception is SNR** ([C19.2](../C19-Audio-Banks/02-snr-spt.md)): its rate field *is* read and
respected, so for SNR you may patch the rate to your audio instead of resampling. This makes SNR the flexible
container for pitch-correct replacement, and ABK/GIN/MUS the "match the original rate" containers.

```python
def apply_rate_rule(audio, container, entry):
    if container == "snr":
        entry.sample_rate = audio.rate      # SNR: patch the rate
        return audio
    return resample(audio, entry.sample_rate)   # ABK/GIN/MUS: resample to the fixed rate
```

## Rule 2 — mono-only codecs

The EA-XA and EA-XAS codecs are **mono per stream** — they encode one channel at a time, carrying a per-channel
predictor history ([C20.3](03-eaxa-decode.md)). Feed a stereo (interleaved L/R) signal into a mono ADPCM slot
and the two channels' samples interleave into the predictor, tangling into **noise**. So you must
**de-interleave stereo to mono** before encoding:

```python
def apply_channel_rule(audio, codec):
    if codec_is_mono(codec):               # EA-XA / EA-XAS
        return to_mono(audio)              # downmix or take one channel — never feed interleaved stereo
    return audio
```

If a sound genuinely needs stereo under a mono codec, it's stored as two mono streams, not one interleaved
stream — so you de-interleave and encode each channel separately.

## Why these rules exist

Both rules follow from the codec design:

- **Fixed rate** — most containers bake the playback rate into how they schedule and mix the sound; the field
  is metadata the mixer set up around, not a live resample instruction. SNR happens to consult it; the others
  don't.
- **Mono predictor** — ADPCM's whole compactness comes from predicting each sample from the *same channel's*
  previous samples. Two channels share one predictor only if interleaved, which ruins the prediction — so the
  codecs are defined mono, and stereo is two streams.

Understanding the *why* makes the rules memorable: rate is baked-in metadata, and ADPCM prediction is
per-channel.

> ✅ *Verified (archive):* the fixed-rate constraint (ABK/GIN/MUS vs SNR) and the mono-only ADPCM requirement
> are confirmed across the shipped audio and the replacement path.

## The combined recipe

Every audio replacement runs both rules before encoding ([C19.6](../C19-Audio-Banks/06-editing-banks.md)):

```python
def prepare_replacement(audio, container, entry):
    audio = apply_rate_rule(audio, container, entry)      # rule 1
    audio = apply_channel_rule(audio, entry.codec)        # rule 2
    return encode(audio, entry.codec)                     # now safe to encode
```

Skip rule 1 and the pitch is wrong; skip rule 2 and it's static. Both are silent failures — the sound still
"plays" — so they're easy to ship by accident and must be caught by decode-and-compare
([C20.6](06-portable-decoder.md)).

## Verifying against the rules

The way to catch a rule violation is to **decode your replacement back to PCM and compare**:

- **Duration mismatch** vs the source ⇒ a rate error (rule 1).
- **Noise / static** where there should be tone ⇒ a stereo-into-mono error (rule 2).
- **Right duration, right character** ⇒ both rules satisfied.

This decode-and-compare is the audio analogue of re-parsing a container to verify an edit — it checks the
*content*, not just that the file loads.

---

### Key takeaways

- **Rule 1 (fixed rate):** resample to the original rate for ABK/GIN/MUS; only SNR lets you patch the rate
  field.
- **Rule 2 (mono-only):** de-interleave stereo to mono before encoding EA-XA/EA-XAS, or channels tangle into
  noise.
- The rules follow from the design: rate is baked-in metadata; ADPCM prediction is per-channel.
- Combined recipe: apply rate rule → apply channel rule → encode.
- Both violations are silent (the sound still plays) — catch them by decoding your replacement and comparing
  duration/character.

**Continue:** [C20.3 — EA-XA v2 decoded](03-eaxa-decode.md) · [Chapter 20 hub](C20-Audio-Codecs.md)
