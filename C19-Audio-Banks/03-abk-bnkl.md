# C19.3 — The ABKC/BNKl Container

> **The one-sentence version:** an `.abk` file is an outer `ABKC` header (file size at `+0x14`, bank offset at
> `+0x20`) that locates an embedded `BNKl` bank (sound count at `+0x06`, a `PT`-offset table at `+0x14`) —
> verified byte-for-byte on the retail front-end bank.

[← C19.2 — SNR routing + SPT payload](02-snr-spt.md) · [Chapter 19 hub](C19-Audio-Banks.md) ·
[Next: C19.4 — The big-endian PT records →](04-pt-records.md)

---

## The two-level container

An ABK is a bank inside a wrapper, and both levels are little-endian on the retail files:

```
ABKC (outer header)
  +0x00  "ABKC"    magic
  +0x04  version   (01 01 01 00)
  +0x14  u32       totalFileSize
  +0x20  u32       sfxBankOffset   → the BNKl bank
  +0x24  u32       sfxBankSize
BNKl (bank, at sfxBankOffset)
  +0x00  "BNKl"    magic
  +0x06  i16       numSounds
  +0x08  i32       bnkSize
  +0x14  i32[N]    PT-offset table (one offset per sound, relative to the bank)
```

The `ABKC` wrapper carries file-level bookkeeping (total size, where the bank is); the `BNKl` bank carries the
sounds. This mirrors the pattern seen throughout the engine — an outer container locating an inner payload,
with a count and an offset table indexing the records ([C16.1](../C16-Scenery-Cull/01-scenery-section.md),
[C8.1](../C8-Geometry-Solids/01-solidlist-container.md)).

## Verified on the retail file

Every field above was read from `SOUND/GLOBAL/FE_COMMON_MB.abk`:

| Field | Value | Check |
|---|---|---|
| `ABKC` magic / version | `ABKC` / `01 01 01 00` | ✓ |
| `+0x14` totalFileSize | 417,928 | = the file's actual size ✓ |
| `+0x20` sfxBankOffset | `0x680` | points at `BNKl` ✓ |
| `+0x24` sfxBankSize | 416,128 | ✓ |
| `BNKl` `+0x06` numSounds | **19** | matches the `PT` table ✓ |
| `BNKl` `+0x08` bnkSize | 416,080 | ✓ |

The `totalFileSize` equalling the real file length and `numSounds` matching the `PT`-offset table are the two
cross-checks that confirm a correct parse — the ABK's version of "the counts agree."

## Reading the bank

```python
def read_abk(buf):
    assert buf[:4] == b"ABKC"
    bank_off = u32(buf, 0x20)
    assert buf[bank_off:bank_off+4] == b"BNKl"
    num = i16(buf, bank_off + 0x06)
    pt_offsets = [i32(buf, bank_off + 0x14 + i*4) for i in range(num)]
    return [parse_pt(buf, bank_off + off) for off in pt_offsets]   # C19.4
```

The `PT` offsets are relative to the bank start, so each sound's record is at `sfxBankOffset + PToffset`. From
there, each `PT` describes one sound ([C19.4](04-pt-records.md)) — and note the switch to **big-endian** for
the `PT` fields, even though the `ABKC`/`BNKl` headers are little-endian.

## Where ABKs are used

The 301 `.abk` files cover the game's self-contained effect banks:

- **Front-end** (`FE_COMMON_MB.abk`) — menu and UI sounds.
- **Engine** (`CAR_00_ENG_MB_EE.abk`, …) — per-car engine sound banks (paired with the GIN engine-sound system,
  [Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).
- **World / effects** — impact, ambience, and gameplay effect banks.

Each is loaded as a unit and its sounds played by id; the ABK keeps a coherent set of related sounds together
with their metadata in one file.

## Editing implications

- **Preserve the two cross-checks.** After editing, `totalFileSize` (`+0x14`) must equal the new file length
  and `numSounds` must match the `PT` table.
- **Fix the offset table on size changes.** Adding/resizing a `PT` shifts later records; re-stamp the
  `PT`-offset table and `sfxBankSize`/`bnkSize` ([C19.6](06-editing-banks.md)).
- **Mind the byte-order switch.** `ABKC`/`BNKl` are little-endian; `PT` fields are big-endian
  ([C19.4](04-pt-records.md)) — a trap when writing.

---

### Key takeaways

- An `.abk` is `ABKC` (outer: file size `+0x14`, bank offset `+0x20`) → `BNKl` (bank: numSounds `+0x06`,
  bnkSize `+0x08`, `PT`-offset table `+0x14`).
- Verified on the retail front-end bank: file-size field = real size, `numSounds` = 19 matching the `PT` table.
- `PT` offsets are relative to the bank; each locates one sound's record.
- ABKs hold self-contained banks (front-end, engine, effects) — 301 of them.
- Preserve the file-size and count cross-checks, fix the offset table on size changes, and respect the
  LE→BE switch at the `PT` level.

**Continue:** [C19.4 — The big-endian PT records](04-pt-records.md) · [Chapter 19 hub](C19-Audio-Banks.md)
