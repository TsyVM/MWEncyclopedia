# Chapter 5 — Textures: the TPK Container Model

> **Goal of this chapter:** open any texture pack in the game, enumerate the textures inside it, locate
> the exact pixel bytes of each, and understand the two on-disk variants well enough to extract and
> replace a texture without breaking the pack.

A **TPK** ("Texture PacK") is the engine's bundle of textures: a container chunk holding a metadata half
(what textures are here, how big, what format) and a data half (the raw pixels). Almost every texture the
game draws — car paint and parts, world surfaces, HUD art, menu atlases — lives inside a TPK. This
chapter is the *container*: how the pack is laid out and how to navigate it. The pixel *encodings* it
holds (DXT, ARGB, palettes, DDS) are [Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md); how a solid
*binds* to a texture is [Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md).

> **Verified against retail data.** Every structural claim here is confirmed by parsing real packs. The
> worked example throughout is `GLOBAL/GLOBALA.BUN`'s pack — internal name `GLOBALMESSAGE`, five textures
> (`MW_LOGO`, `COP_LIGHT`, `FONT_MW_BODY`, `COP_LIGHT_FLASH`, `BASEPOLY`) — whose chunk sizes divide
> exactly by the strides below. One correction to the older record is called out explicitly: the
> per-texture key is **not** a plain Joaat of the short name (see [C5.6](06-the-texture-key.md)).

---

## Deep-dive pages

- [C5.1 — The TPK container anatomy](01-container-anatomy.md): the `0xB3300000` tree — the Info half and the Data half — and how they pair up.
- [C5.2 — The metadata tables](02-metadata-tables.md): the info header, the hash table, the entry table, and the comp-info records, decoded field by field.
- [C5.3 — The two variants: standard vs. compressed](03-two-variants.md): 124-byte entries with one shared pixel blob, versus 24-byte descriptors with per-texture JDLZ blobs — how to tell them apart and why both exist.
- [C5.4 — Finding the pixels](04-finding-pixels.md): walking from a texture's entry to the exact bytes of its mip chain in the data half.
- [C5.5 — Extracting & replacing a texture](05-extract-replace.md): the safe in-place path, the repack path, and the size-tree bookkeeping that keeps a pack loadable.
- [C5.6 — The texture key, honestly](06-the-texture-key.md): what the per-texture 32-bit key is, what it is *not* (a plain name Joaat), and how to work with it regardless.

---

## 5.1 The pack at a glance

A TPK is a container chunk `0xB3300000` whose payload is two sub-containers (with alignment null chunks
between them):

```
0xB3300000  TPKContainer
├── 0x00000000  (null padding)
├── 0xB3310000  TPKInfo            ← the metadata half
│   ├── 0x33310001  TPKInfoHeader  (version, pack name, source path)
│   ├── 0x33310002  TPKHashTable   (one {key, pad} per texture)
│   ├── 0x33310004  TPKEntries     (standard: 124 bytes per texture)   ┐ one or the
│   │   0x33310003  TPKCompEntries (compressed: 24 bytes per texture)  ┘ other variant
│   └── 0x33310005  TPKCompInfo    (32 bytes per texture: dims/format/FourCC)
├── 0x00000000  (null padding)
└── 0xB3320000  TPKData            ← the pixel half
    ├── 0x33320001  TPKDataHeader
    └── 0x33320002  TPKDataRaw     (the raw pixel bytes; DXT/ARGB or per-texture JDLZ blobs)
```

The two halves are parallel: the *N*th entry in the Info half describes the *N*th texture, and its pixels
live in the Data half. The number of textures is the same everywhere — in the worked pack, the hash table
is 40 bytes (5 × 8), the standard entry table is 620 bytes (5 × 124), and the comp-info is 160 bytes
(5 × 32). Those three divisions all yielding **5** is your instant sanity check that you've parsed the
pack correctly. Full anatomy: [C5.1](01-container-anatomy.md).

## 5.2 The metadata, decoded

The **info header** (`0x33310001`, 124 bytes) opens with a `u32` version (5 in retail packs), a 28-byte
pack name at +4 (`GLOBALMESSAGE`), and a 64-byte source path at +0x20
(`Global\GlobalMessageTextures.tpk`) — human-readable strings that immediately tell you what a pack is.

The **hash table** (`0x33310002`) is one `{u32 key, u32 pad}` per texture — a flat index you scan to find
a texture by its key. The **entry table** (`0x33310004`/`0x33310003`) is the per-texture record: name,
key, dimensions, format, and the offset/size of its pixels. The **comp-info** (`0x33310005`, 32 bytes
each) carries the pixel-format descriptor including a FourCC (e.g. `DXT1`/`DXT5`) at +20. Field-by-field
layouts are [C5.2](02-metadata-tables.md).

## 5.3 Two variants

There are two on-disk shapes, distinguished by which entry chunk is present:

- **Standard** (`0x33310004`, 124-byte entries): the Data half is *one* contiguous pixel blob, and each
  entry points at its slice by offset. This is the worked `GLOBALMESSAGE` pack.
- **Compressed** (`0x33310003`, 24-byte descriptors): each texture's pixels are an *individually*
  JDLZ-compressed blob in the Data half, and the descriptor is smaller because per-texture detail lives in
  the comp-info. Common for the shared car templates, where compression saves meaningful space.

Telling them apart is just "which chunk id did I find," and each demands a slightly different pixel-fetch
path. The full comparison and rationale is [C5.3](03-two-variants.md).

## 5.4 From entry to pixels

Given a texture's entry you locate its pixels in `0x33320002`: standard entries give an offset and length
into the single blob; compressed descriptors point at a JDLZ sub-blob you decompress first
([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)). Either way you end with the raw encoded
pixels — DXT blocks or ARGB rows — ready for [Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md) to
decode. [C5.4](04-finding-pixels.md).

## 5.5 Editing safely

Replacing a texture is an application of the size-tree discipline
([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)): if your new pixels are the *same size*
as the old (same dimensions and format), you overwrite in place and touch no size fields — the safest,
most reliable texture mod. If they differ in size, you repack the Data half and fix the ancestor sizes up
to `0xB3300000`. [C5.5](05-extract-replace.md).

---

### Key takeaways

- A TPK is `0xB3300000` = `TPKInfo` (metadata) + `TPKData` (pixels); entries and pixels run in parallel.
- Texture count is consistent across the hash table (×8), entry table (×124 or ×24), and comp-info (×32) —
  divide to sanity-check.
- Two variants: standard (one blob, 124-byte entries) vs. compressed (per-texture JDLZ, 24-byte
  descriptors).
- Same-size replacement is in-place and safe; different-size replacement is a repack with ancestor fixups.
- The per-texture key is **not** a plain Joaat of the short name — treat it as an opaque, stable key
  ([C5.6](06-the-texture-key.md)).

**Next:** [Chapter 6 — Texture Codecs](../C6-Texture-Codecs/C6-Texture-Codecs.md): decoding the DXT and
ARGB pixels this container holds.
