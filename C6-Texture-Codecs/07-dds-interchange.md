# C6.7 — The DDS Interchange Container

> **The one-sentence version:** because MW stores exactly the DXT/ARGB bytes DDS expects, extraction is
> "prepend a 128-byte DDS header" and import is "strip the header, check the format matches, paste the bytes
> back" — a lossless, decode-free round-trip that is the safest way to edit MW textures.

[← C6.6 — Mip chains & sizing](06-mip-chains.md) · [Chapter 6 hub](C6-Texture-Codecs.md) ·
[Next: Chapter 7 — Materials & Texture References →](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)

---

## Why DDS is the right interchange

DDS (`DirectDraw Surface`) is the container the DXT FourCCs and `D3DFORMAT` values were *designed* for. An
MW texture's pixel bytes — DXT blocks or `A8R8G8B8` rows, base level then mip chain — are byte-for-byte what
a DDS file carries in its data section. So moving a texture out of MW and into a paint tool (or back) does
**not** require decoding to RGBA and re-encoding; it requires only attaching or removing a header. That
matters because every decode→edit→encode round-trip through PNG re-quantises the DXT blocks and loses
quality, while a DDS round-trip that leaves the encoded region untouched is bit-exact.

## The 128-byte header

A classic DDS file is a 4-byte magic, a 124-byte `DDS_HEADER`, then the pixel data:

```
+0    u32   magic          = "DDS " (0x20534444)
+4    u32   dwSize         = 124
+8    u32   dwFlags        = CAPS|HEIGHT|WIDTH|PIXELFORMAT (+MIPMAPCOUNT +LINEARSIZE/PITCH)
+12   u32   dwHeight
+16   u32   dwWidth
+20   u32   dwPitchOrLinearSize   (block: bytes of level 0; linear: w*bpp)
+24   u32   dwDepth        = 0
+28   u32   dwMipMapCount  = mip_count
+32   44 bytes reserved
+76   DDS_PIXELFORMAT (32 bytes):
      +0  u32 dwSize       = 32
      +4  u32 dwFlags      = DDPF_FOURCC  (block)  |  DDPF_RGB(|DDPF_ALPHAPIXELS) (ARGB)
      +8  u32 dwFourCC     = "DXT1"/"DXT3"/"DXT5"  (0 for uncompressed)
      +12 u32 dwRGBBitCount= 32   (ARGB)
      +16 u32 dwRBitMask   = 0x00FF0000
      +20 u32 dwGBitMask   = 0x0000FF00
      +24 u32 dwBBitMask   = 0x000000FF
      +28 u32 dwABitMask   = 0xFF000000
+108  u32   dwCaps         = DDSCAPS_TEXTURE (+COMPLEX|MIPMAP if mips)
+112  u32   dwCaps2        = 0
+116  8 bytes reserved
= 128 bytes total, then pixel data
```

The RGBA bit masks above describe `A8R8G8B8` — note `dwRBitMask = 0x00FF0000` and `dwBBitMask = 0x000000FF`,
which is the BGRA-in-memory order from [C6.4](04-argb32.md); DDS states the masks in D3D word terms, so a
correct writer uses exactly these values for MW's `0x15` format.

## Mapping MW formats to DDS

| MW comp-info `+0x14` | DDS pixel format |
|---|---|
| `"DXT1"` | `DDPF_FOURCC`, `dwFourCC = "DXT1"` |
| `"DXT3"` | `DDPF_FOURCC`, `dwFourCC = "DXT3"` |
| `"DXT5"` | `DDPF_FOURCC`, `dwFourCC = "DXT5"` |
| `0x15` ARGB32 | `DDPF_RGB|DDPF_ALPHAPIXELS`, 32 bpp, masks above |
| `0x29` P8 | *no standard DDS equivalent* — convert to a real format first ([C6.5](05-pal8.md)) |

P8 is the one format DDS cannot represent directly (legacy `DDPF_INDEXED` is unsupported by modern tools);
extract P8 by converting to ARGB32 or DXT once you have its palette, rather than trying to DDS-wrap indices.

## Extract: MW → DDS

```python
def mw_to_dds(fmt, w, h, mip_count, encoded_bytes):
    header = build_dds_header(fmt, w, h, mip_count)   # 128 bytes per the layout above
    return header + encoded_bytes                     # encoded_bytes = base + mips, untouched
```

The pixel region is copied verbatim from the entry's byte slice ([C5.4](../C5-Textures-TPK/04-finding-pixels.md)) —
no transformation whatsoever. The resulting `.dds` opens in any DDS-aware editor and shows the exact texture
the game ships.

## Import: DDS → MW

```python
def dds_to_mw(dds_bytes, target_entry):
    fmt, w, h, mips = parse_dds_header(dds_bytes[:128])
    pixels = dds_bytes[128:]
    assert (fmt, w, h) == (target_entry.fmt, target_entry.w, target_entry.h), \
        "format/size changed → this is a repack, not an in-place edit"
    assert len(pixels) == target_entry.total_size, "byte length changed → repack"
    return pixels                          # paste into the blob at target_entry.data_offset
```

Import is where the size-tree discipline reasserts itself. If the incoming DDS matches the target's format,
dimensions, **and** mip count, its pixel length equals `total_size` and you overwrite in place — the safe
tier-1 edit ([C5.5](../C5-Textures-TPK/05-extract-replace.md)). If any of those differ (a bigger DDS, a
format switch, a different mip count), the byte length changes and you must repack the pixel blob and fix
ancestor sizes ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)).

## Practical guidance

- **Match the format on export and re-import.** Export a DXT3 texture as a DXT3 DDS; edit and re-save it as
  DXT3. Let a tool silently convert it to DXT5 and you have changed the format and the size.
- **Preserve mips or regenerate the same count.** The safest in-place edit keeps `mip_count` identical; if
  your editor drops mips, either regenerate them or accept a repack.
- **Watch the ARGB masks.** For `0x15`, write the exact `A8R8G8B8` masks; a writer that emits `R8G8B8A8`
  masks produces a DDS whose red/blue are swapped relative to what MW expects.
- **Don't DDS-wrap P8 indices.** Convert P8 to a real format first ([C6.5](05-pal8.md)).

> ✅ *Verified:* MW's stored DXT/ARGB byte layout (base-first mip chain, block sizes, `A8R8G8B8` order)
> matches the DDS data-section definition, which is why the round-trip is header-only and lossless. The DDS
> header layout is the standard `DDS_HEADER`/`DDS_PIXELFORMAT` structure.

---

### Key takeaways

- DDS carries exactly MW's pixel bytes, so extract = prepend 128-byte header, import = strip header + verify.
- Map DXT1/3/5 to `DDPF_FOURCC`; map `0x15` to 32-bpp `DDPF_RGB|ALPHAPIXELS` with `A8R8G8B8` masks.
- P8 has no direct DDS form — convert to ARGB32/DXT first.
- Same format + size + mip count ⇒ in-place overwrite; any difference ⇒ repack with ancestor fixups.
- The round-trip is bit-exact only if you never decode/re-encode — keep the encoded region untouched.

**Continue:** [Chapter 7 — Materials, Texture References & Animation](../C7-Materials-TexAnim/C7-Materials-TexAnim.md) ·
[Chapter 6 hub](C6-Texture-Codecs.md)
