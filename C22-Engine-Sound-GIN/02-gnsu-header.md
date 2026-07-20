# C22.2 — The Gnsu Header

> **The one-sentence version:** a GIN file opens with `Gnsu`, version `"20"`, then the two floats that define
> everything — **rpmMin at `+0x08`** and **rpmMax at `+0x0C`** — followed by counts, a data offset, and the
> grain-offset table at `+0x20`, all decoded directly from the retail file.

[← C22.1 — GIN is granular synthesis](01-granular-synthesis.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md) ·
[Next: C22.3 — Grains & the grain table →](03-grains.md)

---

## The header, verified

Decoding `GIN_Acura_ITR.gin` byte by byte:

| Offset | Type | Value | Field |
|---|---|---|---|
| `+0x00` | char[4] | `Gnsu` | magic |
| `+0x04` | char[4] | `"20"` | version |
| `+0x08` | `f32` | `2267.0` | **rpmMin** |
| `+0x0C` | `f32` | `8638.1` | **rpmMax** |
| `+0x10` | `u32` | `50` | count (grain/band related) |
| `+0x14` | `u32` | `229` | grain count |
| `+0x18` | `u32` | `162344` | offset to sample/grain data |
| `+0x1C` | `u32` | `32000` | rate / parameter |
| `+0x20` | `u32[]` | `0x1E0, 0xE5D, 0x19AB, 0x2581, …` | **grain-offset table** (increasing) |

Every value here was read straight from the bytes — the floats `2267.0` and `8638.1` at `+0x08`/`+0x0C` are
the RPM bounds, and the monotonically increasing words from `+0x20` are grain offsets
([C22.3](03-grains.md)).

## Correcting the record

An older, community-sourced GIN layout placed `rpmMin`/`rpmMax` at `+0x20`/`+0x24`. That is **wrong** for the
retail `Gnsu` files: at `+0x20` sits the *grain-offset table* (its first entry `0x1E0` = 480, not an RPM), and
the RPM floats are at `+0x08`/`+0x0C`. Reading RPM from `+0x20` would give a nonsense "RPM" of 480 and mistake
the grain table for parameters. This is exactly the kind of claim a reference should verify against bytes
rather than inherit — the retail file is unambiguous.

> ✅ *Verified:* `Gnsu` magic, version `"20"`, `rpmMin = 2267.0` (`+0x08`), `rpmMax = 8638.1` (`+0x0C`), counts
> (`+0x10`/`+0x14`), data offset (`+0x18`), and the increasing grain-offset table (`+0x20`), all from
> `GIN_Acura_ITR.gin`.
> 🟡 *Reasoned:* the precise meaning of the two count fields (`50` vs `229`) and the `+0x1C` parameter (a rate,
> `32000`?) is identified by role; the magic, version, RPM floats, data offset, and grain table are verified.

## The RPM range is the file's identity

The pair `(rpmMin, rpmMax)` = `(2267, 8638)` is the single most important thing in the header, because it
defines the *domain* of the synthesis ([C22.4](04-rpm-bridge.md)):

- **Below rpmMin** — the file doesn't cover it (idle may be a separate grain/handling).
- **Between rpmMin and rpmMax** — the synth maps the current RPM into this range to select and pitch grains.
- **At rpmMax** — the redline; the highest-RPM grains.

Different cars have different ranges — a high-revving Honda reaches ~8600 (as here), a big V8 redlines lower —
and the range in each GIN matches its car's character. Reading the range tells you the engine's rev band at a
glance.

## Reading the header

```python
def read_gin_header(buf):
    assert buf[:4] == b"Gnsu"
    return {
        "version":   buf[4:8].decode("latin1").strip("\x00"),   # "20"
        "rpm_min":   struct.unpack_from("<f", buf, 0x08)[0],
        "rpm_max":   struct.unpack_from("<f", buf, 0x0C)[0],
        "count_a":   u32(buf, 0x10),
        "grain_cnt": u32(buf, 0x14),
        "data_off":  u32(buf, 0x18),
        "param":     u32(buf, 0x1C),
        "grain_offsets": [u32(buf, 0x20 + i*4) for i in range(u32(buf, 0x14))],
    }
```

## Editing implications

- **Preserve the RPM range** unless you intend to re-map the engine's rev band — changing `rpmMin`/`rpmMax`
  shifts where every grain sounds ([C22.4](04-rpm-bridge.md)).
- **Keep the grain table consistent with the data.** If you change grains, the offset table must still point at
  valid grain starts ([C22.3](03-grains.md)).
- **Don't trust the old `+0x20` RPM claim.** Read RPM from `+0x08`/`+0x0C`.
- **Match the version.** `"20"` is the retail version; keep it unless you know a tool expects otherwise.

---

### Key takeaways

- GIN header: `Gnsu` magic, version `"20"`, **rpmMin `+0x08`**, **rpmMax `+0x0C`** (f32), counts, data offset
  `+0x18`, grain-offset table `+0x20`.
- Verified on `GIN_Acura_ITR.gin` (2267–8638 RPM); the older `+0x20`/`+0x24` RPM layout is wrong (that's the
  grain table).
- The `(rpmMin, rpmMax)` pair defines the synthesis domain — the engine's rev band.
- Read the header with the layout above; the grain count (`+0x14`) sizes the offset table.
- Preserve the RPM range and grain-table consistency on edits; read RPM from `+0x08`/`+0x0C`.

**Continue:** [C22.3 — Grains & the grain table](03-grains.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md)
