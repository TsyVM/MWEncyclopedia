# C1.5 — Endianness Islands

> **The one-sentence version:** the whole PC game is little-endian *except* the EA audio stream headers,
> which are big-endian — and knowing exactly where that border sits saves you from reading 44100 Hz as
> 0x44AC0000.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.4 — Alignment & padding](04-alignment-and-padding.md) ·
[Next: C1.6 — Matrices & coordinates →](06-matrices-and-coordinates.md)

---

## What it is

Endianness is the byte order of multi-byte integers. EAGL on PC is **little-endian** by default: the
chunk header, every offset, every count, every float reads low-byte-first. There is one well-defined
exception — the **EA audio stream layer**:

- The `PT`/TLV records inside a sound bank, and
- The `SCHl` header fields of a music stream,

store their integers **big-endian**. These are the "islands": small, sharply bounded regions of
big-endian data inside an otherwise little-endian file.

```c
uint32_t be32(const uint8_t* p){ return (p[0]<<24)|(p[1]<<16)|(p[2]<<8)|p[3]; }
uint16_t be16(const uint8_t* p){ return (uint16_t)((p[0]<<8)|p[1]); }
```

The important word is *island*. It is not that "audio files are big-endian" — the audio *container* is
still an EAGL chunk with a little-endian `{id,size}` header. It is specifically the **stream payload
fields** inside that are big-endian. The same file has both byte orders a few bytes apart.

## How to know which side of the border you're on

The border is **format-defined, not flag-defined** — nothing in the bytes announces "big-endian starts
here." You know by *what you're parsing*:

- Walking the chunk tree, reading geometry, textures, vaults, the world? Little-endian.
- Inside an EA audio container (`ABKC`, `BNKl`, `SCHl`, `MPFF` — the magics
  [Chapter 19/21](../C19-Audio-Banks/C19-Audio-Banks.md) branch on)? The stream TLV fields are
  big-endian.

The fastest sanity check is a known value. A 44.1 kHz sample rate is `0x0000AC44` little-endian. If you
read `0x44AC0000`, you've read a big-endian field with a little-endian reader (or vice versa) — swap and
move on. This "does the sample rate look sane?" test is the standard way to confirm you're on the right
side of the island border. Other good canaries: a channel count should be 1 or 2, not 33 000; a loop
start should be less than the sample count, not billions.

## Why it is designed this way

The EA audio stream format predates and outlives any single platform. It was shared across consoles —
including big-endian CPUs (the PowerPC and MIPS console families EA shipped on) — and EA kept the stream
layer **byte-order-stable across platforms** rather than re-encoding audio per target. The PC build
inherits that cross-platform stream format verbatim, big-endian fields and all, even though the rest of
the PC engine is little-endian. The audio island is, in effect, a fossil of the format's multi-platform
heritage: it was cheaper to byte-swap a handful of header fields on a little-endian host than to
maintain two encodings of every sound.

> 🟡 *Reasoned:* the "console heritage" explanation is the standard account of EA's audio stream format
> and fits the observed big-endian fields; it's an inference about *why*, not a byte-level claim. The
> *fact* that those specific fields are big-endian is ✅ verified against retail data.

## Bending it — mostly "don't," and how to not get bitten

Endianness isn't really something you *bend* for effect — it's something you get *right* or *wrong*. But
the practical consequences are worth spelling out:

**Getting it right:**

- Write your audio parser/writer to **explicitly** byte-swap the stream fields (`be32`/`be16`) rather
  than relying on the host being little-endian. The island is narrow but real; treat it as its own
  reader with its own helpers.
- Keep the boundary mental model crisp: the chunk *header* of an audio container is little-endian (it's
  still an EAGL chunk); the audio *stream payload* fields are big-endian. Bound your swaps to exactly
  the documented big-endian fields — reaching one field too far corrupts the little-endian header.

**Getting it wrong — what you'll see:**

- **Nonsense audio parameters.** A sample rate of `0x44AC0000` (≈ 1.15 billion), a channel count in the
  thousands, a loop point past the end of the file — all are the signature of a flipped read.
- **Silent or garbled playback after a round-trip.** If you decode the header big-endian but write it
  back little-endian (or forget to swap on write), the file parses on your tool and is unreadable to the
  game. Always swap symmetrically: same byte order in and out.

Bottom line: there's no creative upside to flipping endianness — the only move is to model the island
border precisely and swap exactly the fields that need it. Chapters
[19](../C19-Audio-Banks/C19-Audio-Banks.md) and [21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md) enumerate which
fields those are.

---

**Continue:** [C1.6 — Matrices & the Z-up coordinate system](06-matrices-and-coordinates.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
