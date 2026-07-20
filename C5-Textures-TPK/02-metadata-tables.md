# C5.2 — The Metadata Tables

> **The one-sentence version:** four chunks describe a pack's textures — a human-readable info header, a
> flat key table for lookup, a per-texture entry with name/key/dimensions, and a comp-info record whose
> format field names the pixel encoding.

[← C5.1 — Container anatomy](01-container-anatomy.md) · [Chapter 5 hub](C5-Textures-TPK.md) ·
[Next: C5.3 — The two variants →](03-two-variants.md)

---

## The info header — `0x33310001`

124 bytes, and the friendliest chunk in the whole format because it is mostly *text*:

```
+0x00  u32       version        5 in retail packs
+0x04  char[28]  packName       e.g. "GLOBALMESSAGE"
+0x20  char[64]  sourcePath      e.g. "Global\GlobalMessageTextures.tpk"
+0x60  u32       key/hash        pack-level key
...    (remaining fields: counts/flags)
```

Read on the worked pack, these fields yield version `5`, name `GLOBALMESSAGE`, path
`Global\GlobalMessageTextures.tpk` — instantly identifying what the pack is and where it came from in the
build tree. When you are trying to work out which pack holds a texture, the header's `packName` and
`sourcePath` are the first thing to print.

## The hash table — `0x33310002`

One record per texture, `{u32 key, u32 pad}` (8 bytes), so the chunk size divided by 8 is the texture
count. It is a flat index: to find a texture by its key you scan this table for a matching `key` and use
the index to reach the parallel entry and pixels ([C5.1](01-container-anatomy.md)). The `pad` word is a
runtime slot (a resolved pointer/handle at load time); on disk it is zero and you preserve it.

## The entry table — `0x33310004` (standard) / `0x33310003` (compressed)

The per-texture record. In the **standard** variant it is **124 bytes**; in the **compressed** variant a
24-byte descriptor ([C5.3](03-two-variants.md)). The standard entry's field map, from parsing the worked
pack (offsets relative to the entry start):

```
+0x0C  char[24]  name        e.g. "MW_LOGO", "COP_LIGHT_FLASH"
+0x24  u32       key         the per-texture key (see C5.6 — NOT a plain name Joaat)
+0x28  u32       (runtime/handle)
+0x44  u16       width       e.g. 512
+0x46  u16       height      e.g. 128
...    mip count, offset into the pixel blob, and byte length of this texture's pixels
```

The name at +0x0C is a genuine, readable fixed-width string — this is a primary source of texture names
for your resolver ([C2.4](../C2-Identifiers-And-Hashing/04-hash-resolution.md)). The dimensions are 16-bit
(`MW_LOGO` reads 512×128, a sensible logo shape). The pixel offset/length fields are what
[C5.4](04-finding-pixels.md) uses to reach the bytes.

> ✅ name (+0x0C), key (+0x24), and dimensions (+0x44/+0x46) are verified against the retail pack. 🟡 the
> precise offset/length field positions vary in interpretation between pack versions; confirm them for a
> given pack by checking that the offsets tile the data blob without gaps or overlap
> ([C5.4](04-finding-pixels.md)).

## The comp-info — `0x33310005`

32 bytes per texture, and the chunk that names the **pixel format**. The format field sits at **+0x14**
and is either a **FourCC** for a block-compressed format or a **small enum** for an uncompressed one. In
the worked pack the five textures read:

| Texture | format @ +0x14 | Meaning |
|---|---|---|
| `MW_LOGO` | `0x00000015` | uncompressed (enum 21) |
| `COP_LIGHT` | `'DXT3'` | DXT3 / BC2 |
| `FONT_MW_BODY` | `'DXT3'` | DXT3 / BC2 |
| `COP_LIGHT_FLASH` | `0x00000015` | uncompressed (enum 21) |
| `BASEPOLY` | `'DXT3'` | DXT3 / BC2 |

So a single pack freely mixes DXT3 and uncompressed textures, and the comp-info is where you learn which
is which — essential input to the decoder in [Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md). The
FourCC being literally the ASCII `DXT3` (bytes `44 58 54 33`) means you can read it as text; the enum
`0x15` is the engine's own format id for an uncompressed layout.

## The data header — `0x33320001`

24 bytes at the head of the pixel half, before the raw blob. It carries a key and flags for the data
region; for reading pixels you mostly skip it and go to `0x33320002`, but preserve it on a rewrite.

## Why the format lives apart from the entry

Splitting the *format* (comp-info) from the *geometry of the texture* (entry) mirrors the container's
overall description-vs-content split: the entry says "this texture is called X, is 512×128, and its
pixels are here," while the comp-info says "and they are encoded as DXT3." Keeping the format in its own
fixed 32-byte record makes the pixel decoder's job uniform — it reads one comp-info record and knows
exactly how to interpret the bytes the entry points at, regardless of the entry variant.

## Bending it — reading the metadata safely

- **Print the header strings first.** `packName`/`sourcePath` orient you instantly.
- **Register every entry name** in your resolver — these are real, and they light up bare texture-hash
  references elsewhere.
- **Trust comp-info for format, not the entry.** The `+0x14` field (FourCC or enum) is the authority on
  encoding; the entry gives dimensions, the comp-info gives format.
- **Preserve runtime/pad words.** Zero on disk, resolved at load; keep them zero on a rewrite.

---

**Continue:** [C5.3 — The two variants: standard vs. compressed](03-two-variants.md) ·
[Chapter 5 hub](C5-Textures-TPK.md)
