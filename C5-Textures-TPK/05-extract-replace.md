# C5.5 — Extracting & Replacing a Texture

> **The one-sentence version:** extraction is a slice-and-decode; replacement has two tiers — a **same-size
> in-place overwrite** that touches no size field and is nearly foolproof, and a **repack** that rebuilds
> the pixel blob and fixes every ancestor size up the container tree.

[← C5.4 — Finding the pixels](04-finding-pixels.md) · [Chapter 5 hub](C5-Textures-TPK.md) ·
[Next: C5.6 — The texture key, honestly →](06-the-texture-key.md)

---

## Extraction

Extraction never modifies the pack, so it is purely a read. The steps compose the previous pages:

1. Open the pack container `0xB3300000` and split it into the Info half (`0xB3310000`) and Data half
   (`0xB3320000`) ([C5.1](01-container-anatomy.md)).
2. Parse the entry table and comp-info to get, per texture, its name, dimensions, format, `data_offset`,
   `total_size`, and `mip_count` ([C5.2](02-metadata-tables.md), [C5.4](04-finding-pixels.md)).
3. Slice the `0x33320002` blob to get the encoded bytes; if the pack is the compressed variant, JDLZ-inflate
   the sub-blob first ([C5.3](03-two-variants.md)).
4. Decode the bytes to RGBA with the codec chapter ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)),
   or wrap them unchanged in a `.dds` header if you want a byte-exact interchange copy.

Wrapping to DDS is the highest-fidelity extraction because it preserves the exact on-disk encoding (DXT
blocks or ARGB rows) and the full mip chain without ever decoding — the same bytes, just given a header a
paint tool understands. Decoding to PNG is friendlier to edit but discards the encoding decision, so a
round-trip through PNG will re-compress.

## Replacement, tier 1: same-size in-place overwrite

This is the mod you should reach for first because it cannot corrupt the container. If your new texture has
the **same dimensions and the same format** as the old one, its `total_size` is identical, so its bytes
occupy exactly the same slice of the blob. You overwrite those bytes and change **nothing** else — no
offsets, no sizes, no chunk headers.

```python
def replace_in_place(pack_bytes, entry, new_encoded):
    assert len(new_encoded) == entry.total_size, "size changed → use the repack path"
    blob_start = pack_bytes.index_of_chunk_payload(0x33320002)
    lo = blob_start + entry.data_offset
    pack_bytes[lo : lo + entry.total_size] = new_encoded
    return pack_bytes            # every size field untouched — guaranteed loadable
```

Because the size tree ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)) is defined by the
chunk size headers, and you touched none of them, the pack remains structurally valid by construction. The
only requirement is that `new_encoded` really is the same length — same width, same height, same format,
and the **same mip count**, laid out base-first exactly as [C5.4](04-finding-pixels.md) describes.

The practical recipe: export the original, edit it while keeping its dimensions and re-encoding to the
**same** format (if it was DXT3, save DXT3; if ARGB32, save ARGB32), regenerate the same number of mips,
and paste the resulting byte stream back. This is how the overwhelming majority of reliable texture mods
are built.

## Replacement, tier 2: repack with size-tree fixups

If the new texture differs in size — larger dimensions, a different format, more or fewer mips — its byte
length changes, so it no longer fits the old slice. Now you must rebuild the Data half and repair every
size that describes it. This is the general case of the container-editing discipline from
[C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md); applied to a TPK it means:

1. **Rebuild the pixel blob.** Concatenate every texture's bytes in entry order, substituting your new
   texture's bytes for the old. Keep the others byte-identical.
2. **Recompute the offset chain.** Walk the entries in order; each texture's new `data_offset` is the
   running total of all preceding textures' sizes. Only textures *after* the edited one actually move, but
   recomputing the whole chain is simplest and safest.
3. **Update the edited entry's size fields.** Write the new `total_size` (`+0x38`), `base_size` (`+0x40`),
   `width`/`height` (`+0x44`), the packed log-2 dims (`+0x48`) and `mip_count` (`+0x4E`) if any of those
   changed.
4. **Fix the container sizes upward.** The `0x33320002` payload size changes; therefore the size of its
   parent `0x33320000` (Data half), and the grandparent `0xB3300000` (the whole pack), and — critically —
   the size of the **bundle chunk that contains the pack** all change by the same delta. Every ancestor
   size header must be adjusted or the outer file's tree walk desynchronises.
5. **Preserve alignment padding.** If the pack padded the blob to an alignment boundary, re-pad the new blob
   the same way and include that padding in the sizes.

```python
def repack_tpk(entries, new_index, new_bytes):
    entries[new_index].bytes = new_bytes
    entries[new_index].total_size = len(new_bytes)
    # re-tile the blob and recompute offsets
    blob, cursor = bytearray(), 0
    for e in entries:
        e.data_offset = cursor
        blob += e.bytes
        cursor += len(e.bytes)
    blob += align_padding(len(blob))          # match the pack's original alignment
    # caller must now propagate len(blob) delta up: 0x33320002 → 0x33320000
    #                                              → 0xB3300000 → containing bundle chunk
    return blob
```

The step people forget is #4's last clause: the pack lives inside a bundle, and that bundle's chunk header
also states a size. Fix the TPK tree but not the bundle, and the game reads your corrected pack, then
mis-reads whatever chunk follows it. **Size fixups always propagate to the outermost container you are
writing.**

## Choosing a tier

| Situation | Tier |
|---|---|
| Recolor / repaint at identical dimensions and format | **1 — in-place** |
| Swap art but keep the texture's size/format | **1 — in-place** |
| Higher-resolution replacement | 2 — repack |
| Change format (e.g. DXT3 → ARGB32) | 2 — repack |
| Add or remove mips | 2 — repack |

Prefer tier 1 whenever you possibly can; design your edit *around* keeping the size constant. A 512×128
ARGB logo stays a 512×128 ARGB logo; a 256×256 DXT3 font stays 256×256 DXT3. When the size must change,
tier 2 is entirely doable — it is just the full size-tree bookkeeping, and the "offsets must tile the blob"
check from [C5.4](04-finding-pixels.md) is exactly the invariant that tells you the repack is correct.

## Verification after any edit

Re-open the pack you just wrote and confirm three things: (1) the entry offsets still tile the blob
end-to-end with only trailing padding; (2) every ancestor chunk size equals header-plus-children as the
size-tree requires; and (3) the containing bundle still walks cleanly to its end. If all three hold, the
pack is loadable. This re-parse is cheap and catches the two classic failures — an un-propagated size and
an off-by-padding blob — before the game ever sees the file.

---

### Key takeaways

- Extraction = split halves → parse entries → slice (→ JDLZ-inflate if compressed) → decode or DDS-wrap.
- Tier 1 (same size/format/mips) overwrites the slice in place and touches no size field — the safest mod.
- Tier 2 (different size) rebuilds the blob, recomputes the offset chain, and fixes ancestor sizes up to and
  including the containing bundle chunk.
- The most-forgotten fixup is the bundle chunk *outside* the pack; propagate the size delta all the way out.
- Verify by re-parsing: offsets tile the blob, sizes equal header-plus-children, the bundle walks to its end.

**Continue:** [C5.6 — The texture key, honestly](06-the-texture-key.md) · [Chapter 5 hub](C5-Textures-TPK.md)
