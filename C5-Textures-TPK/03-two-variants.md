# C5.3 — The Two Variants: Standard vs. Compressed

> **The one-sentence version:** a pack is either "standard" — 124-byte entries indexing one contiguous
> pixel blob — or "compressed" — 24-byte descriptors each pointing at an individually JDLZ-compressed
> blob; the entry chunk id tells you which, and each needs a different pixel-fetch path.

[← C5.2 — The metadata tables](02-metadata-tables.md) · [Chapter 5 hub](C5-Textures-TPK.md) ·
[Next: C5.4 — Finding the pixels →](04-finding-pixels.md)

---

## What it is

TPK ships in two on-disk shapes. They carry the same *information* (textures with names, formats,
dimensions, and pixels) but organise the pixel half differently, and they are distinguished by which entry
chunk appears in the Info half:

| | Standard | Compressed |
|---|---|---|
| Entry chunk | `0x33310004` | `0x33310003` |
| Entry size | **124 bytes** | **24 bytes** |
| Pixel half | one contiguous blob (`0x33320002`) | many per-texture **JDLZ** blobs |
| Per-texture pixels | a slice of the blob (offset + length) | an individually compressed sub-blob |
| Typical use | HUD/menu/global packs, world | shared car templates |

Detection is trivial: find `0x33310004` → standard; find `0x33310003` → compressed. The worked
`GLOBALMESSAGE` pack is standard (its entries chunk is `0x33310004`, 620 bytes = 5 × 124).

## How each stores pixels

**Standard.** The Data half is a single big blob. Each 124-byte entry contains an **offset** into that
blob and a **byte length**; texture *N*'s pixels are `blob[offset : offset+length]`, already in final
encoded form (DXT blocks or ARGB rows). The engine can hand the whole blob to the GPU and let the entries
describe the sub-regions. Fetching pixels is a slice — no decompression.

**Compressed.** The Data half is a sequence of independently JDLZ-compressed blobs, one per texture. The
24-byte descriptor is smaller because it need only locate its sub-blob; the per-texture format detail
lives in the comp-info ([C5.2](02-metadata-tables.md)). Fetching pixels means locating the sub-blob and
running the JDLZ decompressor ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) to recover the
encoded pixels, which you then decode as usual.

## Why two variants exist

The split is a space-vs-simplicity trade the build pipeline makes per pack:

- **Standard** favours *load simplicity and contiguous upload*. One blob, direct slices, no per-texture
  decompression — ideal for packs read constantly or streamed as a unit (HUD, world surfaces).
- **Compressed** favours *disk footprint* where it pays most: the car system reuses large template
  textures across many vehicles, and compressing each independently saves meaningful space, at the cost of
  a decompression step when a texture is needed. Independent per-texture blobs also let the engine
  decompress only the textures it actually uses, rather than the whole pack.

> 🟡 *Reasoned:* the "which packs use which variant, and why" mapping is inferred from where each variant
> appears (global/HUD → standard; car templates → compressed) and the obvious space/simplicity trade; the
> *structural* difference (chunk id, entry size, per-texture JDLZ) is ✅ verified.

## Handling both in one reader

A robust TPK reader branches once, at the entry chunk, and normalises to the same in-memory texture list:

```python
def read_tpk(info_chunks, data_blob):
    if has_chunk(info_chunks, 0x33310004):        # standard
        entries = parse_entries_124(info_chunks[0x33310004])
        for e in entries:
            e.pixels = data_blob[e.offset : e.offset + e.length]
    elif has_chunk(info_chunks, 0x33310003):      # compressed
        descs = parse_descriptors_24(info_chunks[0x33310003])
        for d in descs:
            sub = data_blob[d.offset : d.offset + d.comp_length]
            d.pixels = jdlz_decompress(sub) if sub[:4] == b'JDLZ' else sub
    # from here, both paths yield encoded pixels + comp-info format → Chapter 6
```

The point is to *converge*: after the branch, both variants present the same thing — encoded pixels plus a
format from comp-info — so the decoder ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) and the
editor ([C5.5](05-extract-replace.md)) never need to know which variant they came from.

## Bending it — variant-aware editing

- **Detect by chunk id, never by extension or size alone.** `0x33310004` vs `0x33310003` is the definitive
  test.
- **Keep the variant when you rewrite.** Converting a compressed pack to standard (or vice versa) is
  possible but changes every offset and the whole Data half; unless you have a reason, preserve the
  variant the game shipped.
- **For compressed packs, remember pixels are two layers deep** — decompress the sub-blob *then* decode the
  format. Skipping the JDLZ step feeds DXT garbage to your decoder.

---

**Continue:** [C5.4 — Finding the pixels](04-finding-pixels.md) · [Chapter 5 hub](C5-Textures-TPK.md)
