# C21.2 — The SCHl/SCCl/SCDl/SCEl Blocks

> **The one-sentence version:** each MUS section is built from EA-stream blocks — `SCHl` (header with a `GSTR`
> format sub-block, big-endian fields), optional `SCCl` (continuation), `SCDl` (data), and `SCEl` (end and
> DWORD-align) — and the header's big-endian format is the classic byte-order trap.

[← C21.1 — MUS: EA-stream sections](01-mus-sections.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md) ·
[Next: C21.3 — Loop points & interactive sections →](03-loops-sections.md)

---

## The four blocks

A section is a run of tagged EA-stream blocks, verified present in `MW_Music.mus`:

| Tag | Role | Notes |
|---|---|---|
| `SCHl` | section **header** | format in a `GSTR` sub-block; **big-endian** fields |
| `SCCl` | **continuation** | optional, for multi-part headers |
| `SCDl` | **data** | the compressed audio payload |
| `SCEl` | **end** | finalises the section, advances to the next DWORD boundary |

The order is `SCHl` (→ `SCCl`?) → `SCDl` (one or more) → `SCEl`. The header describes the section; the data
blocks carry the audio; the end block closes and aligns it ([C21.1](01-mus-sections.md)).

## SCHl and the GSTR format

The `SCHl` header opens the section and, in the retail file, contains a **`GSTR`** ("GStream") sub-block that
holds the stream **format** — the codec tag, sample rate, and channel count:

```
SCHl  (tag)
  size
  GSTR  (format sub-block)
    codec tag   (0x00 PCM, 0x02 EA-ADPCM, 0x0A EA-XA, 0xEA EA-MP3)
    sample rate, channels   ← BIG-ENDIAN
    loop points             (C21.3)
```

The codec tag selects the decoder ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)); the rate and
channels set the playback format. These format fields are **big-endian** — read them little-endian and you get
nonsense (44,100 Hz becomes garbage), the identical trap to ABK `PT` records
([C19.4](../C19-Audio-Banks/04-pt-records.md)).

## SCDl carries the audio

`SCDl` blocks hold the section's **compressed audio** in the codec the header named. A section may have one or
several `SCDl` blocks (streamed in parts); concatenate their payloads and decode with the header's codec to
recover the section's PCM. This is where the shared codec layer does the work — the MUS just frames the encoded
bytes.

## SCEl ends and aligns

`SCEl` finalises the section and — critically — advances to the next **DWORD boundary**
([C21.1](01-mus-sections.md)). This alignment is what keeps the *next* section's `SCHl` starting where the
walker expects. Skip or mis-handle `SCEl`'s alignment and every section after this one is misread — the
music-section version of the padding-desync failure ([C1.4](../C1-EAGL-Container-Model/04-alignment-and-padding.md)).

## Parsing a section

```python
def parse_section(buf, p):
    assert buf[p:p+4] == b"SCHl"
    header = parse_schl(buf, p)               # GSTR: codec, rate, channels (BIG-ENDIAN), loops
    p = header.end
    data = b""
    while buf[p:p+4] in (b"SCCl", b"SCDl"):
        blk = parse_block(buf, p)
        if blk.tag == b"SCDl": data += blk.payload
        p = blk.end
    assert buf[p:p+4] == b"SCEl"
    end = align_dword(parse_block(buf, p).end)
    return Section(header, data, end)
```

> ✅ *Verified:* `SCHl`/`SCCl`/`SCDl`/`GSTR` blocks are present in `MW_Music.mus` with the MUS magic; the
> big-endian format fields and codec-tag set are the confirmed EA-stream model.
> 🟡 *Reasoned:* the exact byte offsets of rate/channels within `GSTR` are the format's detail; the block model,
> big-endian convention, and codec tags are verified.

## Editing implications

- **Write format fields big-endian.** Patching the header means byte-swapping rate/channels — LE writes corrupt
  it ([C19.4](../C19-Audio-Banks/04-pt-records.md)).
- **Keep the block order and terminator.** `SCHl` → data → `SCEl`; a missing `SCEl` breaks alignment for every
  later section.
- **Preserve DWORD alignment.** `SCEl` must land the next section on a 4-byte boundary
  ([C21.6](06-editing-music.md)).
- **Match the codec.** Re-encode a replaced section with the header's codec, or update the codec tag to match
  your audio ([C20.1](../C20-Audio-Codecs/01-codec-set.md)).

---

### Key takeaways

- A section = `SCHl` (header) → optional `SCCl` → `SCDl` (data) → `SCEl` (end/align).
- `SCHl` carries a `GSTR` format sub-block with the codec tag, rate, and channels — **big-endian** fields.
- Codec tags: `0x00` PCM, `0x02` EA-ADPCM, `0x0A` EA-XA, `0xEA` EA-MP3 — decode with Chapter 20.
- `SCEl` finalises and DWORD-aligns; mis-handling it desyncs every later section.
- Edit format fields big-endian, keep block order and the `SCEl` terminator, and preserve alignment.

**Continue:** [C21.3 — Loop points & interactive sections](03-loops-sections.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md)
