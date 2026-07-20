# C8.1 — The SolidList Container & Directory

> **The one-sentence version:** a SolidList (`0x80134000`) opens with a header block holding three tables —
> a list header with the object count and source filename, a **sorted** hash table for binary-search lookup,
> and an object directory mapping each name-hash to a file offset and size — followed by the solids
> themselves.

[← Chapter 8 hub](C8-Geometry-Solids.md) · [Next: C8.2 — The object header →](02-object-header.md)

---

## The header block

Every SolidList begins with a header container `0x80134001` that carries three parallel tables. Parsed from
`CARS/COBALTSS/GEOMETRY.BIN` (316 objects):

```
0x80134001  header block
├── 0x00134002  list header      144 bytes
├── 0x00134003  hash table       2528 bytes  = 316 × 8
└── 0x00134004  object directory 7584 bytes  = 316 × 24
```

### List header (`0x00134002`, 144 bytes)

The list header names the bundle and counts its contents:

- `+0x0C` — **object count** (`0x13C` = 316).
- `+0x10` — **source filename** as ASCII (`GEOMETRY.BIN`), the original build-time file name.
- A `DEFAULT` string follows — the fallback material/name used when a solid has no more specific one.

The embedded filename is a gift for reverse-engineering: it tells you what the bundle *was* at build time,
even after it has been packed and renamed.

### Hash table (`0x00134003`, N × 8 bytes)

Each entry is `{u32 name-hash, u32 pad}`. The table has one entry per object, and — critically — it is
**sorted in ascending name-hash order**, not object-storage order. Verified: all 316 entries are
monotonically non-decreasing, and `COBALTSS_BASE_A`'s hash `0x54DF8EF4` is present in the table even though
it is the *first* object stored but not the first table entry. Sorting is what enables **binary search** by
hash ([C8.6](06-lookup.md)); a linear list would not need to be sorted.

### Object directory (`0x00134004`, N × 24 bytes)

Each entry is the locator for one object:

```
+0x00  u32  name-hash
+0x04  u32  file offset   (absolute offset of this object's 0x80134010 chunk)
+0x08  u32  size          (byte size of the object)
+0x0C  u32  size          (repeated)
+0x10  u32  0
+0x14  u32  0
```

Verified: directory entry 0's offset (`0x139A00`) points at a chunk whose id is exactly `0x80134010` — a
Solid. So the directory maps *name-hash → where the object lives and how big it is*, which is how the engine
(and you) jump to an object without walking the whole list.

## The three-way count check

The object count appears in three independent places, and they must agree:

```
list header +0x0C        = 316
hash table size / 8      = 2528 / 8  = 316
object directory / 24    = 7584 / 24 = 316
```

If these ever disagree, your parse is wrong or the bundle is damaged — this is the fastest structural
sanity check for a SolidList, the geometry analogue of the TPK's "three tables all divide to N"
([C5.1](../C5-Textures-TPK/01-container-anatomy.md)).

## Why three tables?

The split is a classic space/speed design:

- The **hash table** is small (8 bytes/entry) and sorted, so a lookup is a `log₂(N)` binary search over a
  compact array — cache-friendly and fast.
- The **directory** (24 bytes/entry) holds the heavier locators (offset, size); you only touch it *after*
  the hash search has found the index.
- The **list header** carries the one-time metadata (count, source name).

So a lookup binary-searches the tiny sorted hash array, then reads the matching directory row to get the
offset. Separating the search key from the payload keeps the hot path in the smallest possible footprint.

## A container reader

```python
def read_solidlist_header(hdr_chunks):
    lh   = hdr_chunks[0x00134002]
    n    = u32(lh, 0x0C)
    fname = cstr(lh, 0x10)
    htab = hdr_chunks[0x00134003]        # n × 8, sorted by hash
    dirt = hdr_chunks[0x00134004]        # n × 24
    assert len(htab)//8 == n == len(dirt)//24, "object-count mismatch"
    hashes = [u32(htab, i*8) for i in range(n)]
    directory = [(u32(dirt,i*24), u32(dirt,i*24+4), u32(dirt,i*24+8)) for i in range(n)]
    return n, fname, hashes, directory     # (hash, offset, size) rows
```

> ✅ *Verified:* object count 316 agrees across all three tables; hash table is sorted ascending and contains
> `COBALTSS_BASE_A`'s hash; directory entry 0's offset lands on a `0x80134010` chunk. Source filename
> `GEOMETRY.BIN` is embedded in the list header.

---

### Key takeaways

- SolidList header block `0x80134001` holds: list header (`0x00134002`), sorted hash table (`0x00134003`),
  object directory (`0x00134004`).
- Object count lives at list-header `+0x0C` and is cross-checked by `hashtable/8` and `directory/24`.
- The hash table is **sorted ascending** for binary search; storage order is independent of table order.
- The directory maps `name-hash → {offset, size}`; entry offsets point at `0x80134010` solids.
- Lookups binary-search the compact hash array, then read the directory — search key separated from payload.

**Continue:** [C8.2 — The object header, field by field](02-object-header.md) · [Chapter 8 hub](C8-Geometry-Solids.md)
