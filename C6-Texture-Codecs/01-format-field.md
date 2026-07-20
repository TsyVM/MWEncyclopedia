# C6.1 — The Format Field & MW's Format Set

> **The one-sentence version:** comp-info `+0x14` holds the format — a numeric `D3DFORMAT` when its top
> three bytes are zero (`0x15` ARGB32, `0x29` P8), or a FourCC otherwise (`"DXT1"`, `"DXT3"`, `"DXT5"`) —
> and that one field is the only branch a decoder needs.

[← Chapter 6 hub](C6-Texture-Codecs.md) · [Next: C6.2 — DXT1 / BC1 →](02-dxt1-bc1.md)

---

## Where the format lives

Each texture has a 32-byte comp-info record in `0x33310005`, in the same order as the entry table. The
format descriptor sits at `+0x14`. Reading it correctly means recognising two encodings in one field:

- If `bytes[+0x15..+0x17]` are all zero, the field is a **little-endian `u32` `D3DFORMAT` enum value**.
- Otherwise, the four bytes are a **FourCC** — a literal four-character code (`D,X,T,1`, …) that a `u32`
  read would show as `0x31545844` but which you should treat as ASCII.

```python
def read_format(compinfo_rec):
    fc = compinfo_rec[0x14:0x18]
    if fc[1:] == b"\x00\x00\x00":
        return ("d3dfmt", fc[0])          # e.g. 0x15, 0x29
    return ("fourcc", fc.decode("ascii")) # e.g. "DXT1", "DXT3", "DXT5"
```

The rest of the record is stable across retail data and does not affect decoding: `+0x0C = 5` and
`+0x10 = 6` are constants, `+0x08` is a small per-texture flag (0 or 1), and the remaining words are zero.
Do not read dimensions from comp-info — width and height come from the **entry** at `+0x44`
([C5.2](../C5-Textures-TPK/02-metadata-tables.md)), and comp-info only classifies the pixel encoding.

## The five formats, and nothing else

Censusing every comp-info record in every uncompressed pack yields exactly five distinct formats. That
closed set is what any MW texture tool must handle — and all it must handle:

| Tag | Name | Meaning | Block/texel size |
|---|---|---|---|
| `"DXT1"` | BC1 | RGB (+1-bit alpha) block compression | 8 B per 4×4 block |
| `"DXT3"` | BC2 | BC1 color + explicit 4-bit alpha | 16 B per 4×4 block |
| `"DXT5"` | BC3 | BC1 color + interpolated 8-bit alpha | 16 B per 4×4 block |
| `0x15` (21) | `D3DFMT_A8R8G8B8` | uncompressed 32-bit BGRA-in-memory | 4 B per texel |
| `0x29` (41) | `D3DFMT_P8` | 8-bit palette index | 1 B per texel |

These D3D enum values are the standard Direct3D 9 `D3DFORMAT` constants — `A8R8G8B8 = 21`, `P8 = 41` — which
confirms the numeric readings are genuine D3D formats and not an MW-private numbering. The FourCCs are the
same ones DirectDraw and every DDS file use, which is exactly why DDS round-trips so cleanly
([C6.7](07-dds-interchange.md)).

## Branching a decoder

A complete MW texture decoder is a five-way switch on this field, feeding the base-plus-mip byte slice from
[C5.4](../C5-Textures-TPK/04-finding-pixels.md):

```python
def decode_texture(fmt, w, h, encoded_bytes):
    kind, val = fmt
    if kind == "fourcc":
        if val == "DXT1": return decode_dxt1(w, h, encoded_bytes)   # C6.2
        if val == "DXT3": return decode_dxt3(w, h, encoded_bytes)   # C6.3
        if val == "DXT5": return decode_dxt5(w, h, encoded_bytes)   # C6.3
    else:
        if val == 0x15:   return decode_argb32(w, h, encoded_bytes) # C6.4
        if val == 0x29:   return decode_p8(w, h, encoded_bytes)     # C6.5
    raise ValueError(f"unexpected MW texture format {fmt!r}")
```

Because the set is closed and small, the `raise` at the bottom is a genuine tripwire: if you ever hit it on
retail data, you have mis-parsed comp-info (usually by reading the wrong record or misaligning the table),
not found a sixth format.

## Why the distribution looks the way it does

The counts (DXT3 ≫ DXT1 ≫ DXT5 > P8 > ARGB32) reflect deliberate art-pipeline choices:

- **DXT3 dominates** because the open-world art is saturated with cutouts, decals, signage and foliage that
  need clean *hard-edged* alpha. DXT3's explicit 4-bit alpha has no interpolation artifacts on sharp edges,
  and at 16 B/block it is still a 4:1 win over ARGB32.
- **DXT1** carries fully opaque surfaces (road, terrain, building shells) at 8 B/block — the cheapest option.
- **DXT5** appears where alpha is *smooth* (soft glows, gradients) and interpolation helps.
- **P8 and ARGB32** are rare specialists: P8 for small indexed HUD/sprite art, ARGB32 where a texture must
  be pixel-exact and uncompressed (logos, fonts, lookup textures).

> ✅ *Verified:* the format field location (`+0x14`), the dual FourCC/numeric encoding, and the exact
> five-format set with their counts, all from parsing retail comp-info records.

---

### Key takeaways

- Format is at comp-info `+0x14`; zero top-three-bytes ⇒ numeric `D3DFORMAT`, else a FourCC.
- The complete MW format set is DXT1, DXT3, DXT5, `0x15` (ARGB32), `0x29` (P8) — five, closed.
- Numeric values are standard D3D9 enums (`A8R8G8B8=21`, `P8=41`); FourCCs match DDS exactly.
- Dimensions come from the entry (`+0x44`), never comp-info; comp-info only classifies the encoding.
- A decoder is a five-way switch on `+0x14`; hitting the default on retail data means a parse error, not a
  new format.

**Continue:** [C6.2 — DXT1 / BC1, block by block](02-dxt1-bc1.md) · [Chapter 6 hub](C6-Texture-Codecs.md)
