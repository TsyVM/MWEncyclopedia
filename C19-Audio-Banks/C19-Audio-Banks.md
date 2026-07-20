# Chapter 19 — Audio: Banks (SNR/SPT/ABK)

> **Goal of this chapter:** open the game's sound banks — the SNR/SPT routing-plus-payload pair and the
> ABK/BNKl container of big-endian `PT` records — enumerate the sounds inside, and understand the routing that
> turns a game event into a played sample.

Most Wanted's sound effects — the world ambience, the UI clicks, the crashes, the cop chatter cues — live in
**banks**. A bank is a container of encoded audio plus the metadata that routes and describes each sound. This
chapter decodes the two bank shapes and their records; the codecs that turn the bytes into samples are
[Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md), and music and engine sound get their own chapters
([21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md), [22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).

> **Verified against retail data.** The ABK container is confirmed byte-for-byte on `SOUND/GLOBAL/FE_COMMON_MB.abk`:
> magic `ABKC`, `totalFileSize` at `+0x14` = 417,928 (the exact file size), `sfxBankOffset` at `+0x20` =
> `0x680`, and at that offset a `BNKl` bank whose `+0x06` `numSounds` = **19** and `+0x08` `bnkSize` = 416,080,
> with a `PT`-offset table following. The 301 `.abk` files in `SOUND/` all share this shape.

---

## Deep-dive pages

- [C19.1 — Two bank shapes](01-two-shapes.md): SNR/SPT (routing + payload) versus ABK/BNKl (container of `PT`
  records), and when each is used.
- [C19.2 — SNR routing + SPT payload](02-snr-spt.md): the routing table, the 32-byte entry, and why SNR is the
  one bank whose sample rate you may patch.
- [C19.3 — The ABKC/BNKl container](03-abk-bnkl.md): the outer `ABKC` header and the embedded `BNKl` bank,
  decoded from the retail file.
- [C19.4 — The big-endian PT records](04-pt-records.md): the `PT` TLV that describes each sound — and the
  byte-order trap.
- [C19.5 — SFX vocabulary & routing](05-vocabulary-routing.md): how sounds are named and how an event finds its
  sample.
- [C19.6 — Editing banks safely](06-editing-banks.md): replacing a sound within the fixed-rate and
  fit-the-span constraints.

---

## 19.1 Two shapes for two jobs

Sounds come in two bank layouts, and recognising which you have is step one:

- **SNR + SPT** — a **routing table** (`.SNR`) paired with a **payload blob** (`.SPT`). The SNR lists sounds
  with their metadata and points into the SPT for the actual audio. Notably, SNR is the **one** bank whose
  sample-rate field you may safely edit ([C19.2](02-snr-spt.md)).
- **ABK / BNKl** — an outer **`ABKC`** container wrapping an embedded **`BNKl`** bank of **`PT`** records, each
  `PT` a big-endian TLV describing one sound ([C19.3](03-abk-bnkl.md), [C19.4](04-pt-records.md)). The 301
  `.abk` files (effects, engine, front-end) are this shape.

Both ultimately do the same thing — hold encoded audio plus per-sound metadata — but organise it differently,
and each has its own editing rules ([C19.6](06-editing-banks.md)).

## 19.2 SNR/SPT: routing and payload split

The SNR/SPT pair separates *what sounds exist and how to play them* (SNR) from *the audio bytes* (SPT). Each
SNR entry (32 bytes) carries an id, a name offset, the sound's offset and size in the SPT, its codec tag,
sample rate, channel count, and duration ([C19.2](02-snr-spt.md)). To play a sound the engine reads its SNR
entry, seeks into the SPT, and decodes with the entry's codec. SNR's editable sample-rate field makes it the
friendliest bank for pitch-correct replacement.

## 19.3 ABK/BNKl: container and bank

The ABK is a two-level container, verified on the retail file:

```
ABKC (outer)      +0x14 totalFileSize   +0x20 sfxBankOffset   +0x24 sfxBankSize
  └── BNKl (bank) +0x06 numSounds (19)  +0x08 bnkSize         +0x14 PT-offset table
        └── PT, PT, PT …  (one big-endian TLV per sound)
```

The `ABKC` header locates the `BNKl` bank; the `BNKl` header counts the sounds and points at each one's `PT`
record via an offset table ([C19.3](03-abk-bnkl.md)). Everything checks: the header's file size equals the
actual file, and the bank's `numSounds` matches its `PT` table.

## 19.4 PT records are big-endian

Each sound in a `BNKl` bank is a **`PT`** record — a **tag-length-value** structure whose fields are stored
**big-endian**, unlike almost everything else in the game ([C19.4](04-pt-records.md)). The `PT` describes the
sound: its codec, sample rate, loop points, and the location of its audio data. The byte-order trap is real —
read a `PT`'s rate field little-endian and you get nonsense, the same failure as reading `SCHl` music headers
wrong ([C21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)).

---

### Key takeaways

- Sound effects live in **banks**: SNR/SPT (routing + payload) and ABK/BNKl (container of `PT` records).
- ABK is verified: `ABKC` → `BNKl` (numSounds, bnkSize, PT-offset table), file-size field matching the file.
- SNR/SPT splits routing (32-byte entries: id, offsets, codec, rate, duration) from the SPT payload; SNR's rate
  field is uniquely editable.
- `PT` records are **big-endian** TLVs describing each sound — read them in the right byte order.
- The codecs that decode the audio are [Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md).

**Next:** [Chapter 20 — Audio Codecs](../C20-Audio-Codecs/C20-Audio-Codecs.md): the EA-XA/XAS/ADPCM/MP3 math
that turns bank bytes into samples.
