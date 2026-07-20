# C5.4 — Finding the Pixels

> **The one-sentence version:** each standard entry stores a **byte offset** and a **total size** into the
> single `0x33320002` pixel blob; texture *N*'s bytes are `blob[offset : offset + size]`, laid out as a
> base level followed by its mip chain — and the five offsets in the worked pack tile the blob end-to-end
> with zero gaps.

[← C5.3 — The two variants](03-two-variants.md) · [Chapter 5 hub](C5-Textures-TPK.md) ·
[Next: C5.5 — Extracting & replacing →](05-extract-replace.md)

---

## The three numbers that locate a texture

Everything you need to pull a texture's pixels lives in its 124-byte entry (`0x33310004`). Three fields do
the work, and this book verified their offsets by parsing `GLOBAL/GLOBALA.BUN` (pack `GLOBALMESSAGE`,
five textures) and confirming the results tile the pixel blob exactly:

| Entry offset | Field | Meaning |
|---|---|---|
| `+0x30` | `data_offset` (u32) | byte offset of this texture's pixels **within** the `0x33320002` blob |
| `+0x38` | `total_size` (u32) | total bytes of the mip chain (base level **plus** all smaller mips) |
| `+0x40` | `base_size` (u32) | bytes of the base level (mip 0) alone |
| `+0x44` | `width` (u16), `height` (u16) | base-level dimensions |
| `+0x48` | `log2w`, `log2h`, flags (bytes) | packed log-2 dimensions and format flags |
| `+0x4E` | `mip_count` (u16) | number of levels in the chain |

The blob itself is the payload of `0x33320002` in the Data half (`0xB3320000`). So the full path is:

```
pixels(N) = payload(0x33320002) [ entry[N].data_offset : entry[N].data_offset + entry[N].total_size ]
```

No pointer chasing, no indirection table — the offset is a direct index into one contiguous buffer. That
is the whole point of the standard variant: the engine can upload the blob and let entries carve out
sub-regions.

## The worked pack, tiled exactly

Here are all five textures of `GLOBALMESSAGE`, decoded from their entries and laid against the blob. The
`format` column comes from the comp-info FourCC ([C5.2](02-metadata-tables.md)); `0x15` is
`D3DFMT_A8R8G8B8` (uncompressed 32-bit ARGB), `DXT3` is the block-compressed format decoded in
[Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md).

| # | Name | W×H | Format | `data_offset` | `total_size` | ends at |
|---|------|-----|--------|--------------:|-------------:|--------:|
| 0 | `MW_LOGO`         | 512×128 | ARGB32 (`0x15`) | `0x000000` | `0x040000` (262144) | `0x040000` |
| 1 | `COP_LIGHT`       | 128×32  | DXT3            | `0x040000` | `0x001500` (5376)   | `0x041500` |
| 2 | `FONT_MW_BODY`    | 256×256 | DXT3            | `0x041500` | `0x010000` (65536)  | `0x051500` |
| 3 | `COP_LIGHT_FLASH` | 128×128 | ARGB32 (`0x15`) | `0x051500` | `0x015500` (87296)  | `0x066A00` |
| 4 | `BASEPOLY`        | 32×32   | DXT3            | `0x066A00` | `0x000400` (1024)   | `0x066E00` |

Each texture's `data_offset` is exactly the previous texture's end. The chain closes at `0x066E00`
(421 376 bytes); the `0x33320002` payload is 421 496 bytes, so the final 120 bytes are alignment padding.
This "offsets tile the blob" property is your single most powerful correctness check: **if the offsets and
sizes don't chain end-to-end, you've misread a field or the pack is corrupt.**

Notice the sizes cross-check the dimensions and format:

- `MW_LOGO`, 512×128 ARGB32 → 512 × 128 × 4 = **262 144** = `total_size` (no mips: `base_size` equals
  `total_size`).
- `FONT_MW_BODY`, 256×256 DXT3 → DXT3 is 1 byte per texel, so 256 × 256 = **65 536** = `total_size`.
- `BASEPOLY`, 32×32 DXT3 → 32 × 32 = **1024** = `total_size`.

When `total_size > base_size`, the extra bytes are the mip chain (each level a quarter of the previous),
and `mip_count` at `+0x4E` tells you how many levels to expect. `COP_LIGHT_FLASH` is the mip-bearing case:
128×128 ARGB32 base = 65 536, plus five smaller mips = 87 296 total.

## The base-plus-mips byte layout

Within a texture's slice, levels are stored **largest first**, packed with no padding between them:

```
[ base level (base_size bytes) ][ mip 1 ][ mip 2 ] ... [ mip N ]
        width×height              w/2×h/2   w/4×h/4       1×1-ish
```

For a block format (DXT), each level's byte size is `max(1, ceil(w/4)) * max(1, ceil(h/4)) * blockBytes`
(8 bytes/block for DXT1, 16 for DXT3/DXT5), and levels never shrink below one 4×4 block. For a linear
format (ARGB32, `0x15`), it's simply `w * h * 4`, halving `w` and `h` each level down to 1×1. To seek to
mip *k*, sum the sizes of levels `0 … k-1` and add that to `data_offset`.

## From bytes to a decoder

Once you have the slice, the format FourCC from comp-info decides what the bytes mean, and
[Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md) turns them into RGBA. The division of labour is
clean and worth internalising:

- **This container (C5)** answers *where are the bytes and how big is each level* — offsets, sizes, the
  base-plus-mips layout.
- **The codec chapter (C6)** answers *what do the bytes mean* — DXT block decode, ARGB channel order,
  palette expansion.

A minimal, variant-agnostic fetch that returns encoded bytes plus the format tag:

```python
def fetch_texture(entry, compinfo, data_blob):
    off, total = entry.data_offset, entry.total_size
    encoded = data_blob[off : off + total]          # base + mips, largest first
    return {
        "name":   entry.name,
        "w":      entry.width,
        "h":      entry.height,
        "mips":   entry.mip_count,
        "format": compinfo.fourcc,                  # 0x15 → ARGB32, b"DXT3" → DXT3, ...
        "bytes":  encoded,                          # hand to Chapter 6's decoder
    }
```

For the **compressed variant** ([C5.3](03-two-variants.md)) the only change is that `encoded` is a JDLZ
sub-blob you decompress first ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)); after that step
the base-plus-mips layout and the format tag behave identically.

## Common ways this goes wrong

- **Confusing `base_size` with `total_size`.** If you copy only `base_size` bytes you lose every mip and
  the GPU samples garbage at distance. Use `total_size` to copy the whole chain; use `base_size` only when
  you deliberately want mip 0.
- **Assuming DXT is 4 bits/texel like DXT1.** DXT3/DXT5 are **1 byte/texel** (16 bytes per 4×4 block).
  Only DXT1 is 8 bytes/block. Sizing a DXT3 texture as DXT1 halves your byte count and desynchronises every
  later offset.
- **Reading dimensions from the wrong field.** Width/height are a u16 pair at `+0x44`, *not* the packed
  log-2 bytes at `+0x48`. The log-2 bytes (`9,7` for 512×128) are a convenience/validation copy, not the
  raw dimensions.
- **Forgetting the tail padding.** The blob can be a little larger than the sum of texture sizes; that
  trailing padding is normal and must be preserved on repack ([C5.5](05-extract-replace.md)).

---

### Key takeaways

- Three entry fields locate pixels: `data_offset` (`+0x30`), `total_size` (`+0x38`), `base_size` (`+0x40`),
  all indexing the single `0x33320002` blob.
- The five worked offsets tile the blob end-to-end with only trailing padding — the definitive parse check.
- Each slice is base level first, then the mip chain, packed with no inter-level padding.
- `total_size` = whole chain; `base_size` = mip 0. Copy `total_size` to preserve mips.
- The container gives you *where and how big*; [Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md) gives
  you *what the bytes mean*.

**Continue:** [C5.5 — Extracting & replacing a texture](05-extract-replace.md) ·
[Chapter 5 hub](C5-Textures-TPK.md)
