# C19.4 — The Big-Endian PT Records

> **The one-sentence version:** each sound in a `BNKl` bank is a `PT` record — a tag-length-value structure
> whose fields are stored **big-endian** — describing the sound's codec, sample rate, loop points, and the
> location of its audio, so reading it little-endian yields nonsense.

[← C19.3 — The ABKC/BNKl container](03-abk-bnkl.md) · [Chapter 19 hub](C19-Audio-Banks.md) ·
[Next: C19.5 — SFX vocabulary & routing →](05-vocabulary-routing.md)

---

## The TLV structure

A `PT` record is a **tag-length-value** stream: a sequence of typed fields, each a small tag followed by a
value, describing one sound. Rather than a fixed struct, it's a self-describing list of properties — codec,
sample rate, channel count, loop start/end, and a pointer to the encoded audio. A reader walks the tags until
it hits the terminator, collecting the properties it recognises.

```python
def parse_pt(buf, off):
    props = {}
    p = off
    while True:
        tag = buf[p]                          # field tag
        if tag == PT_END: break
        value, p = read_be_value(buf, p+1)    # BIG-ENDIAN value
        props[tag] = value
    return props        # {codec, sampleRate, channels, loopStart, loopEnd, dataOffset, ...}
```

The exact tag vocabulary is the format's detail; the structural point is that a `PT` is a **property list**,
extensible and variable-length, which is why sounds with different needs (looping vs one-shot, mono vs stereo)
coexist in one bank.

## The big-endian trap

Here is the single most important fact about `PT` records: **their multi-byte fields are big-endian**, even
though the enclosing `ABKC`/`BNKl` headers ([C19.3](03-abk-bnkl.md)) — and nearly everything else in the game —
are little-endian. This is a deliberate EA convention for the `PT`/`SC*` audio-metadata family, and it is a
classic trap:

- Read a `PT`'s sample-rate field **little-endian** and you get a wildly wrong number (44,100 Hz = `0xAC44`
  becomes `0x44AC00…` garbage).
- The same trap bites the music `SCHl` headers ([C21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)), which are
  also big-endian.

So a `PT` reader must switch to big-endian for the value bytes. The `ABKC`/`BNKl` scaffolding stays
little-endian; the `PT` payload flips. Miss the switch and every sound's metadata is nonsense.

> ✅ *Verified (archive):* `PT` records are big-endian TLVs describing each sound; the byte-order convention is
> confirmed and shared with the `SC*` music family.

## What a PT describes

The properties a `PT` carries are exactly what you need to extract and play the sound:

- **Codec** — which decoder ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)) turns the bytes into
  samples.
- **Sample rate / channels** — the playback format.
- **Loop points** — for sustained sounds (engine loops, ambience).
- **Data location / size** — where the encoded audio lives (in the bank or a paired payload) and how big.

With these, extraction is: read the `PT`, find the audio, decode with the codec at the stated rate.

## Editing implications

- **Write fields big-endian.** When you patch a `PT` (e.g. loop points), byte-swap — LE writes corrupt it.
- **Rate is fixed on replace.** Like ABK/GIN/MUS generally ([C19.6](06-editing-banks.md)), a `PT`'s rate is not
  honoured as a resample instruction — resample your replacement to the original rate rather than patching the
  field and expecting a pitch change ([C20.2](../C20-Audio-Codecs/02-replacement-rules.md)).
- **Preserve the TLV terminator.** A `PT` reader walks tags to the end marker; a malformed record without a
  proper terminator runs off into the next sound.
- **Keep loop points sane.** A loop that ends past the audio, or starts after it ends, clicks or drifts.

---

### Key takeaways

- Each sound is a `PT` **TLV** record — a self-describing property list (codec, rate, channels, loop points,
  data location).
- `PT` fields are **big-endian** — the enclosing `ABKC`/`BNKl` are little-endian; the `PT` payload flips.
- Reading a `PT` little-endian yields garbage rates/offsets — the classic audio byte-order trap (shared with
  `SCHl`).
- A `PT` gives everything to extract and play a sound: codec, format, loops, data location.
- Edit `PT` fields big-endian, resample rather than rely on the fixed rate field, and keep the terminator and
  loop points valid.

**Continue:** [C19.5 — SFX vocabulary & routing](05-vocabulary-routing.md) · [Chapter 19 hub](C19-Audio-Banks.md)
