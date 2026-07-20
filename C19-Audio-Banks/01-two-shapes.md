# C19.1 — Two Bank Shapes

> **The one-sentence version:** sounds ship in two bank layouts — SNR/SPT (a routing table plus a payload
> blob) and ABK/BNKl (an `ABKC` container wrapping a `BNKl` bank of big-endian `PT` records) — and the first
> step with any bank is recognising which shape you have.

[← Chapter 19 hub](C19-Audio-Banks.md) · [Next: C19.2 — SNR routing + SPT payload →](02-snr-spt.md)

---

## Same job, two organisations

Both bank shapes hold the same two things — **encoded audio** and **per-sound metadata** (codec, rate, loop
points, location) — but package them differently:

| | SNR / SPT | ABK / BNKl |
|---|---|---|
| Structure | routing table (`.SNR`) + payload (`.SPT`) | `ABKC` container → `BNKl` bank → `PT` records |
| Metadata | 32-byte SNR entries | big-endian `PT` TLVs |
| Byte order | little-endian | `PT` fields **big-endian** |
| Rate field | **editable** (unique) | fixed (don't patch) |
| Typical use | streamed / routed effects | effect, engine, front-end banks (301 `.abk`) |

Recognising the shape tells you how to parse it, what byte order to use, and which editing rules apply
([C19.6](06-editing-banks.md)).

## Detecting the shape

Detection is by magic:

```python
def detect_bank(buf):
    if buf[:4] == b"ABKC":                 # ABK container
        return "abk"
    if buf[:4] == b"NSFR" or is_snr_magic(buf):   # SNR routing table
        return "snr"
    # SPT is the payload half of an SNR pair — identified by its partner .SNR
    ...
```

The `.abk` files announce themselves with `ABKC`; the SNR/SPT pair is two files (`.SNR` routing, `.SPT`
payload) that belong together. Never assume — read the magic, because the byte-order and editing rules differ.

## Why two shapes exist

The split reflects two ways sounds are used:

- **SNR/SPT** suits sounds accessed by a **routing lookup** — the SNR is a table you index to find and describe
  a sound, with the SPT as a shared payload pool. Its editable rate field makes it the flexible option.
- **ABK/BNKl** suits **self-contained banks** of related effects — a front-end bank, an engine bank — loaded as
  a unit. The `ABKC`→`BNKl`→`PT` nesting keeps a bank's sounds together with their metadata in one file.

Both sit on the **shared codec layer** ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)): whichever shape
holds a sound, the audio inside is one of the same handful of codecs, so decoding either bank reuses the same
decoders.

## The shared codec layer underneath

The crucial simplification is that banks are *containers*, not codecs. A bank routes and describes; the audio
bytes are EA-XA, EA-XAS, IMA-ADPCM, or EA-MP3 ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)). So the
work splits cleanly:

- **This chapter** — find a sound in a bank and read its metadata (codec, rate, offset, size).
- **Chapter 20** — decode the bytes with the codec the metadata names.

Master the bank shapes here and the codecs there, and you can extract, decode, and replace any sound in the
game.

## Editing implications preview

The two shapes carry different constraints, detailed in [C19.6](06-editing-banks.md):

- **SNR** — you may patch the sample rate, so pitch-correct replacement is easiest here.
- **ABK/BNKl and the rest** — the rate field is **not** patched on replace, so you must **resample** your
  replacement to the original rate first, or it plays at the wrong pitch/speed.

Knowing the shape tells you immediately which rule you're under.

---

### Key takeaways

- Two bank shapes: **SNR/SPT** (routing table + payload) and **ABK/BNKl** (`ABKC`→`BNKl`→`PT` records).
- Detect by magic (`ABKC`, SNR magic); byte order and editing rules differ by shape.
- SNR/SPT suits routed/streamed sounds (editable rate); ABK/BNKl suits self-contained banks (fixed rate).
- Both sit on the shared codec layer — the bank routes/describes, [Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)
  decodes.
- The shape determines the replacement rule: SNR can patch rate; ABK/others require resampling first.

**Continue:** [C19.2 — SNR routing + SPT payload](02-snr-spt.md) · [Chapter 19 hub](C19-Audio-Banks.md)
