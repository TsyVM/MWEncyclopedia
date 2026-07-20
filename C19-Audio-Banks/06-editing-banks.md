# C19.6 — Editing Banks Safely

> **The one-sentence version:** replace a sound by encoding your audio with the entry's codec and fitting it to
> the original's constraints — resample to the fixed rate for ABK/GIN/MUS (patch the rate only for SNR),
> de-interleave stereo to mono for ADPCM, and fit the byte span so no offsets move.

[← C19.5 — SFX vocabulary & routing](05-vocabulary-routing.md) · [Chapter 19 hub](C19-Audio-Banks.md) ·
[Next: Chapter 20 — Audio Codecs →](../C20-Audio-Codecs/C20-Audio-Codecs.md)

---

## The two replacement rules

Audio replacement is governed by two rules that apply across every bank
([C20.2](../C20-Audio-Codecs/02-replacement-rules.md)):

1. **Fixed rate fields.** ABK, GIN, and MUS do **not** honour a patched sample-rate field — they play the
   replacement at the *original* rate. So you must **resample your audio to the original rate first**, or it
   plays at the wrong pitch/speed. **SNR is the exception** — its rate field *is* respected, so you may patch
   it instead of resampling ([C19.2](02-snr-spt.md)).
2. **Mono-only codecs.** EA-XA / EA-XAS encode one channel at a time
   ([C20.3](../C20-Audio-Codecs/03-eaxa-decode.md)); feeding stereo into a mono ADPCM slot interleaves the
   channels into noise. **De-interleave stereo to mono** before encoding.

Get these two right and the rest of replacement is bookkeeping.

## The safe recipe

For a bank sound (ABK `PT` or SNR entry):

```python
def replace_sound(entry, new_wav):
    audio = load_wav(new_wav)
    if bank_is_fixed_rate(entry):          # ABK / GIN / MUS
        audio = resample(audio, entry.sample_rate)   # rule 1: match the original rate
    else:                                  # SNR
        entry.sample_rate = audio.rate               # or patch the rate
    if codec_is_mono(entry.codec):         # EA-XA / EA-XAS
        audio = to_mono(audio)             # rule 2: de-interleave
    encoded = encode(audio, entry.codec)
    fit_to_span(encoded, entry.size)       # avoid an offset cascade if you can
    write_payload(entry, encoded)
```

`fit_to_span` is the key to a clean edit: if the re-encoded audio fits the original byte length (trim or pad to
match), no offset in the bank moves and the edit is a pure overwrite — the audio version of a same-size texture
swap ([C5.5](../C5-Textures-TPK/05-extract-replace.md)).

## Same-size vs resize

- **Same-size (preferred).** Encode to fit the original `size`; overwrite in place; no offset table or size
  field changes. The most reliable audio edit.
- **Different-size (repack).** If the audio can't fit the span, rewrite the payload and re-stamp every later
  sound's offset, plus the bank/file size fields and the cross-checks ([C19.3](03-abk-bnkl.md)). For ABK that's
  the `PT`-offset table + `bnkSize` + `ABKC` `totalFileSize`; for SNR/SPT it's the SPT offsets
  ([C19.2](02-snr-spt.md)).

## Byte-order and format traps

Two mistakes corrupt banks silently:

- **Writing `PT`/`SCHl` fields little-endian.** They're big-endian ([C19.4](04-pt-records.md)) — byte-swap on
  write.
- **Encoding stereo into a mono slot.** Rule 2 — de-interleave first, or the channels tangle into static.

Both produce audio that "plays" but is wrong (garbage metadata or noise), so they're easy to ship by accident.

## Verifying an audio edit

Because a bad audio edit often still "plays," verification must go beyond "does it load":

1. **Metadata reads back correctly** — codec, rate, offsets in the right byte order
   ([C19.4](04-pt-records.md)).
2. **Duration and level match the source** — decode your replacement back to PCM and compare length and RMS
   against the original audio; a mismatch means a rate or channel error
   ([C20.2](../C20-Audio-Codecs/02-replacement-rules.md)).
3. **Cross-checks hold** — file-size and count fields agree after a resize ([C19.3](03-abk-bnkl.md)).
4. **Listen** — the decisive test: the sound plays at the right pitch, length, and without static.

The decode-and-compare in step 2 is the audio analogue of re-parsing a container — it catches the wrong-rate
and wrong-channel errors that the file structure can't.

---

### Key takeaways

- Two rules: resample to the **fixed rate** (ABK/GIN/MUS; SNR may patch instead) and **de-interleave stereo to
  mono** for ADPCM.
- Safe recipe: match rate → to mono → encode → **fit to the original span** → overwrite (no offset cascade).
- Same-size edits overwrite in place; resizes re-stamp offsets and the file-size/count cross-checks.
- Watch the traps: write `PT`/`SCHl` fields **big-endian**, never encode stereo into a mono slot.
- Verify by reading metadata back, decode-and-compare duration/level, checking cross-checks, and listening.

**Continue:** [Chapter 20 — Audio Codecs](../C20-Audio-Codecs/C20-Audio-Codecs.md) · [Chapter 19 hub](C19-Audio-Banks.md)
