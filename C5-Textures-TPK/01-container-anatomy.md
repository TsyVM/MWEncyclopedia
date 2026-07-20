# C5.1 — The TPK Container Anatomy

> **The one-sentence version:** a texture pack is one container chunk split into a metadata half and a
> pixel half, whose per-texture records run in lockstep — parse both halves, pair them by index, and the
> pack is open.

[← Chapter 5 hub](C5-Textures-TPK.md) · [Next: C5.2 — The metadata tables →](02-metadata-tables.md)

---

## What it is

A TPK is a plain EAGL chunk tree ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) whose
root is `0xB3300000`. Because it is an ordinary chunk tree, your universal dumper already prints its shape
— nothing here needs a TPK-specific parser to *navigate*, only to *interpret*. Dumped, the worked
`GLOBALMESSAGE` pack looks like this (sizes are real):

```
[C] 0xB3300000 TPKContainer      size=422776  @0x0
    [ ] 0x00000000 (null pad)    size=48      @0x8
    [C] 0xB3310000 TPKInfo       size=976     @0x40
        [ ] 0x33310001 InfoHeader   size=124   @0x48
        [ ] 0x33310002 HashTable    size=40    @0xCC     (5 × 8)
        [ ] 0x33310004 Entries      size=620   @0xFC     (5 × 124, standard variant)
        [ ] 0x33310005 CompInfo     size=160   @0x370    (5 × 32)
    [ ] 0x00000000 (null pad)    size=96      @0x418
    [C] 0xB3320000 TPKData       size=421624  @0x480
        [ ] 0x33320001 DataHeader   size=24    @0x488
        [ ] 0x00000000 (null pad)   size=80    @0x4A8
        [ ] 0x33320002 DataRaw      size=421496 @0x500
```

## The two halves

The design cleanly separates *description* from *content*:

- **`0xB3310000` TPKInfo — the metadata half.** Everything *about* the textures: the pack's name and
  source path, a hash table for lookup, one entry per texture with its name/key/dimensions/format, and a
  comp-info table with the pixel-format descriptor. This half is small and is what you read to enumerate a
  pack.
- **`0xB3320000` TPKData — the pixel half.** The raw encoded pixels: in the standard variant, one large
  contiguous blob (`0x33320002`, here 421,496 bytes) that all entries index into; in the compressed
  variant, a series of per-texture JDLZ blobs. This half is nearly the whole file by size.

The null chunks (`0x00000000`) between the sub-containers are alignment filler
([C1.4](../C1-EAGL-Container-Model/04-alignment-and-padding.md)) — step over them; they carry no meaning
but they *are* counted in the parent's size.

## The parallelism invariant

The single most important structural fact: **the metadata records and the pixel data are parallel arrays
indexed by the same texture number.** Texture *N*'s hash-table slot, entry, and comp-info record all sit
at index *N*, and its pixels are the *N*th slice (or *N*th sub-blob) in the Data half. This is why the
three metadata chunks all divide by the same texture count — 40/8, 620/124, 160/32 all give 5 — and it is
your fastest correctness check: if those divisions disagree, you have mis-identified a chunk or a stride.

```python
def texture_count(tpk):
    n_hash    = hashtable_size // 8      # {u32 key, u32 pad}
    n_entry   = entries_size // stride   # stride = 124 (standard) or 24 (compressed)
    n_comp    = compinfo_size // 32
    assert n_hash == n_entry == n_comp, "mis-parse: counts disagree"
    return n_hash
```

## Why split metadata from pixels

Two engineering reasons, both visible in the layout:

1. **Cheap enumeration.** The engine (and your tool) can read the entire Info half — kilobytes — to know
   what a pack contains, without touching the multi-megabyte pixel blob. Browsing, validating, and
   building lookup tables all happen against the small half.
2. **Contiguous GPU upload.** In the standard variant, the pixel half is one contiguous blob, so the
   textures can be streamed to the GPU as a block with entries describing where each starts — the same
   in-place residency logic as everywhere else ([C1.12](../C1-EAGL-Container-Model/12-runtime-view.md)).
   Separating description from content is what makes that contiguity possible.

## Bending it — navigate before you interpret

- **Dump first.** The TPK is a chunk tree; your generic dumper reveals its structure before you write a
  line of TPK code. Confirm the two halves and the null pads are where this page says.
- **Check the count three ways.** Hash-table/8, entries/stride, comp-info/32 must agree. A disagreement is
  a mis-parse, caught in one line.
- **Respect the null pads.** They're counted bytes; a repack must reproduce the alignment they provide
  ([C5.5](05-extract-replace.md)).

---

**Continue:** [C5.2 — The metadata tables](02-metadata-tables.md) · [Chapter 5 hub](C5-Textures-TPK.md)
