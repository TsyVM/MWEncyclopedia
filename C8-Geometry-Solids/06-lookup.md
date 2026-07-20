# C8.6 — Finding an Object: Binary Search

> **The one-sentence version:** because the hash table (`0x00134003`) is sorted ascending by name-hash, you
> find a solid in `log₂(N)` steps — binary-search the hash to get its index, then read that row of the
> directory (`0x00134004`) for the file offset and size, and seek straight to the object.

[← C8.5 — The placement transform](05-transform.md) · [Chapter 8 hub](C8-Geometry-Solids.md) ·
[Next: C8.7 — Editing solids safely →](07-editing.md)

---

## Why the table is sorted

The SolidList keeps two parallel arrays: the compact **hash table** (8 bytes/entry) and the heavier
**directory** (24 bytes/entry). The hash table is sorted in ascending name-hash order — verified across all
316 entries of the worked bundle — for one reason: **binary search**. A sorted array of 32-bit keys lets the
engine locate any object in `⌈log₂ N⌉` comparisons (nine for 316 objects) over a cache-friendly block,
instead of scanning. That is the payoff for storing the table sorted rather than in object order.

Crucially, **table order ≠ storage order**. `COBALTSS_BASE_A` is the first object *stored* in the file, but
its hash `0x54DF8EF4` sits wherever it falls in ascending order — near the middle of the table, not the
front. The two orders are independent; the directory row that the hash search lands on carries the offset
that jumps to wherever the object actually lives.

## The lookup, end to end

```python
def find_solid(name_hash, hashes, directory):
    # hashes: sorted list of 316 name-hashes (from 0x00134003)
    # directory: list of (hash, offset, size) rows (from 0x00134004), same index space
    lo, hi = 0, len(hashes) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        h = hashes[mid]
        if   h == name_hash: 
            return directory[mid]           # (hash, offset, size) → seek to offset
        elif h <  name_hash: lo = mid + 1
        else:                hi = mid - 1
    return None                              # not in this SolidList
```

Two subtleties matter for a correct implementation:

- **The directory must be indexed the same way as the hash table.** In practice the directory is also keyed
  by hash (each row starts with the name-hash), so after the binary search you can either index the directory
  by the found position or, more robustly, confirm `directory[mid].hash == name_hash`. Verified: directory
  row 0's hash matches its hash-table counterpart and its offset (`0x139A00`) lands on a `0x80134010` chunk.
- **Unsigned comparison.** Name-hashes are unsigned 32-bit; compare them as unsigned or a high-bit hash will
  sort wrong and the search will miss.

## From offset to object

The directory row gives an **absolute file offset** and a **size**. Seek there and you are at the object's
`0x80134010` chunk; parse its header ([C8.2](02-object-header.md)) and mesh container
([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)) from that point. The size lets you bound the read (and skip
the object wholesale if you only want to enumerate).

```python
def load_solid(buf, name, hashes, directory):
    row = find_solid(asset_hash(name_or_read_it), hashes, directory)
    if row is None: raise KeyError(name)
    _, offset, size = row
    return parse_solid(buf[offset : offset + size])
```

Note the input is a **name-hash**, not a name — you either already have the hash (from a reference elsewhere)
or you read the stored hash. Because the asset hash is not generally computable from the name
([C8.3](03-object-hash.md)), lookups in practice flow from hash to object, and the ASCII name is used only to
display what you found.

## Enumerating vs. searching

Two access patterns, two tables:

- **Enumerate all objects** — walk the SolidList's `0x80134010` children in storage order (or walk the
  directory rows). This is what you do to list or batch-process every solid.
- **Find one object** — binary-search the sorted hash table, then read the directory. This is what the engine
  does at runtime when something references a solid by hash.

Both are O(N) to build the tables once; searching is O(log N) per query thereafter.

> ✅ *Verified:* the hash table is sorted ascending (all 316 entries); the directory maps hash → offset/size
> and its offsets point at `0x80134010` solids; binary search over the sorted keys is therefore correct.

---

### Key takeaways

- The hash table is sorted ascending specifically to allow **binary search** — `log₂ N` lookups.
- Table order is independent of storage order; the directory's offset bridges the two.
- Lookup: binary-search the hash → directory row → absolute offset + size → seek to the `0x80134010` object.
- Compare hashes as **unsigned**; confirm the found directory row's hash matches.
- Enumerate via storage order / directory rows; search via the sorted hash table.

**Continue:** [C8.7 — Editing solids safely](07-editing.md) · [Chapter 8 hub](C8-Geometry-Solids.md)
