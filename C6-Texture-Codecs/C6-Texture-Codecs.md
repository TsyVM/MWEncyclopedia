# Chapter 6 — Texture Codecs

> **Goal of this chapter:** take the raw encoded bytes a TPK hands you ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md))
> and turn them into pixels — and back again — for every format the game actually ships: the DXT block
> codecs, uncompressed ARGB, palettized P8, their mip chains, and the DDS container you round-trip through.

Chapter 5 answered *where the bytes are and how big each level is*. This chapter answers *what the bytes
mean*. The two are deliberately separate: the container never needs to understand a pixel, and the codec
never needs to understand a chunk. Once you can slice a texture's base-plus-mip byte stream and read its
format tag from comp-info (`+0x14`), everything here is pure pixel math.

> **Verified against retail data.** The format set below is not assumed — it is the census of every
> comp-info record in every uncompressed pack in the game. Five formats appear, and only five:

| comp-info `+0x14` | Format | D3D name | Bytes | Count (packs scanned) |
|---|---|---|---|---:|
| `"DXT3"` | DXT3 / BC2 | (FourCC) | 1 B/texel (16 B/4×4 block) | **4454** |
| `"DXT1"` | DXT1 / BC1 | (FourCC) | ½ B/texel (8 B/4×4 block) | **3409** |
| `"DXT5"` | DXT5 / BC3 | (FourCC) | 1 B/texel (16 B/4×4 block) | **73** |
| `0x29` (41) | P8 | `D3DFMT_P8` | 1 B/texel (8-bit index) | **58** |
| `0x15` (21) | ARGB32 | `D3DFMT_A8R8G8B8` | 4 B/texel | **9** |

The distribution tells a story on its own: **DXT3 is the game's workhorse** — it pairs BC1 color with a
cheap, artifact-free explicit-alpha block, which suits the world's countless cutout and decal textures.
DXT1 covers opaque surfaces at half the size. DXT5, P8, and ARGB32 are the specialist minority.

---

## Deep-dive pages

- [C6.1 — The format field & MW's format set](01-format-field.md): reading the FourCC/D3DFMT tag from
  comp-info, the five formats, and how to branch a decoder on it.
- [C6.2 — DXT1 / BC1, block by block](02-dxt1-bc1.md): the two 565 endpoints, the interpolated 4-color
  palette, the 2-bit selectors, and the punch-through-alpha special case.
- [C6.3 — DXT3 & DXT5 alpha (BC2 / BC3)](03-dxt3-dxt5.md): explicit 4-bit alpha versus interpolated 8-point
  alpha, and why MW leans on DXT3.
- [C6.4 — Uncompressed ARGB32](04-argb32.md): channel order, row layout, and the BGRA-in-memory trap.
- [C6.5 — Palettized P8](05-pal8.md): 8-bit indices, the 256-entry palette, and how MW stores the index
  data.
- [C6.6 — Mip chains, sizing & the block rule](06-mip-chains.md): level sizes for linear and block formats,
  the 4×4 floor, and seeking to mip *k*.
- [C6.7 — The DDS interchange container](07-dds-interchange.md): wrapping MW bytes in a DDS header for
  byte-exact extraction, and importing edited DDS back into a pack.

---

## 6.1 The one field that decides everything

Every texture's comp-info record (`0x33310005`, 32 bytes) carries its format at `+0x14`. If bytes
`+0x15..+0x17` are zero, `+0x14` is a **numeric** `D3DFORMAT` (`0x15` = ARGB32, `0x29` = P8); otherwise the
four bytes are a **FourCC** (`"DXT1"`, `"DXT3"`, `"DXT5"`). That single branch selects the decoder. The
surrounding record fields are constant in retail data (`+0x0C = 5`, `+0x10 = 6`) and a per-texture flag at
`+0x08`; none of them change the decode. Full treatment: [C6.1](01-format-field.md).

## 6.2 The DXT family in one idea

All three DXT formats share one compression trick: split the image into independent **4×4 texel blocks**
and, per block, store a tiny local palette plus per-texel selectors. For color, that palette is two 16-bit
RGB **endpoints** (5-6-5 bits) and two interpolated midpoints; each texel spends **2 bits** choosing one of
the four. That is the whole of DXT1's color, and it is the color half of DXT3 and DXT5 too. Sixteen texels
× 2 bits = 32 bits of selectors + 32 bits of endpoints = **8 bytes per color block** ([C6.2](02-dxt1-bc1.md)).

The formats differ only in **alpha**:

- **DXT1** — no separate alpha block; one bit of "punch-through" transparency stolen from the endpoint
  ordering. 8 bytes/block total.
- **DXT3** — a full **4-bit explicit alpha** per texel, stored as an extra 8-byte block *before* the color
  block. 16 bytes/block. No interpolation, so no alpha-gradient artifacts — the reason MW prefers it.
- **DXT5** — an **interpolated** alpha block: two 8-bit alpha endpoints and 3-bit selectors into an 8-value
  ramp. 16 bytes/block. Best for smooth alpha, slightly worse for hard edges.

[C6.3](03-dxt3-dxt5.md) works both alpha blocks bit by bit.

## 6.3 The uncompressed formats

**ARGB32** (`0x15`) is the simple case: four bytes per texel, row after row, no blocks and no mips-in-blocks
math. The only trap is channel order — on disk and in a D3D `A8R8G8B8` surface the bytes are **B, G, R, A**
in memory (little-endian ARGB), which you must swizzle to RGBA for most modern tools ([C6.4](04-argb32.md)).

**P8** (`0x29`) stores one **8-bit palette index** per texel; the base level is exactly `width × height`
bytes (verified — `TREAD`, 128×64, base = 8192 = 128·64). The 256-entry color table is resolved by the
engine at load time (the comp-info's on-disk palette pointer is zeroed), so P8 is the one format where the
container alone does not hand you self-contained color — [C6.5](05-pal8.md) covers what is and isn't
recoverable from the file.

## 6.4 Mips and the block floor

Within a texture's byte slice the levels are stored largest-first with no padding
([C5.4](../C5-Textures-TPK/04-finding-pixels.md)). Level sizes follow the format's byte rule, halving `w`
and `h` each step — but **block formats never shrink below one 4×4 block**: a 2×2 or 1×1 DXT level still
costs a full block (8 or 16 bytes). Forgetting that floor is the most common way a mip walk desynchronises.
[C6.6](06-mip-chains.md) gives the exact per-level formulas and a seek-to-mip routine.

## 6.5 Getting bytes in and out: DDS

The cleanest way to move a texture between MW and a paint tool is the **DDS** container, because its pixel
payload is *exactly* the DXT/ARGB bytes MW already stores — extraction is "prepend a 128-byte header," and
import is "strip the header, verify the format matches, paste the bytes back" ([C5.5](../C5-Textures-TPK/05-extract-replace.md)).
No decode, no re-compress, no quality loss. [C6.7](07-dds-interchange.md) gives the header field-by-field
and the format↔`DDS_PIXELFORMAT` mapping.

---

### Key takeaways

- MW ships exactly five texture formats: **DXT3 (dominant), DXT1, DXT5, ARGB32 (`0x15`), P8 (`0x29`)** —
  verified by census, not assumption.
- The format tag lives at comp-info `+0x14`: zero high bytes ⇒ numeric `D3DFORMAT`, else a FourCC.
- All DXT color is the same 8-byte block: two 565 endpoints, four interpolated colors, 2-bit selectors.
  DXT3/DXT5 add a 16-byte total by prefixing an alpha block (explicit 4-bit vs interpolated).
- ARGB32 is B,G,R,A in memory; P8 is 8-bit indices with an engine-resolved palette.
- Block formats floor at one 4×4 block per level; DDS is a header-only, lossless interchange for all of it.

**Next:** [Chapter 7 — Materials, Texture References & Animation](../C7-Materials-TexAnim/C7-Materials-TexAnim.md):
how a solid actually *binds* one of these textures by its key.
