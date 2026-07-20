# C30.2 — The String-Table Format

> **The one-sentence version:** every localization file is the same chunk (`0x00039000`) with a header of
> offsets to an id-keyed index and a string pool — so reading a string is: look up its id in the index, follow
> the offset into the pool.

[← C30.1 — The label system](01-label-system.md) · [Chapter 30 hub](C30-Localization-Labels.md) ·
[Next: C30.3 — Labels.bin →](03-labels.md)

---

## One format for all

Both `Labels.bin` and the per-language files use the **same** on-disk format — verified by identical header
words. The file is a single EAGL chunk `0x00039000` whose body is a small header pointing at sub-tables:

```
+0x00  0x00039000     chunk id
+0x04  size
+0x08  0x10           header size / table count
+0x0C  0x11A8         offset → (sub-table A: index / hash)
+0x10  0x1814         offset → (sub-table B)
+0x14  0xA554         offset → string pool
+0x18  0x100          (count / flags)
```

The header is a directory of **offsets to sub-tables**, the same organising idea as VPAK
([C11.1](../C11-Attribute-Vaults/01-vpak-header.md)) and CARP
([C18.1](../C18-Road-Network-CARP/01-carp-format.md)): a fixed header locating variable-size tables. That the
same header words appear in `Labels.bin` and `English.bin` is the proof they share this format.

## Index + string pool

The two structural pieces are an **index** (keyed by id) and a **string pool** (the text bytes):

- **The index** maps a **string id** to an **offset** into the pool — the id-keyed lookup stage-3 uses
  ([C30.1](01-label-system.md)). It may be a hash table (id → slot) or a sorted array for binary search, like
  the geometry hash table ([C8.6](../C8-Geometry-Solids/06-lookup.md)).
- **The string pool** is the concatenated null-terminated (or length-prefixed) strings — the actual text.

Reading a string:

```python
def read_string(table, string_id):
    offset = table.index[string_id]          # id → pool offset
    return read_pooled_string(table.pool, offset)   # the text at that offset
```

For `Labels.bin` the "strings" are the label names ([C30.3](03-labels.md)); for a language file they are the
translated text ([C30.4](04-language-files.md)). Same structure, different content.

## Why hash/index + pool

Separating an id-keyed index from a string pool is the standard string-table design, and it buys:

- **Fast lookup.** Resolving an id is an index probe, not a scan — important because the UI resolves labels
  constantly.
- **Compact ids.** The UI carries small ids/offsets, not strings; the bulky text lives once in the pool.
- **Shared structure across languages.** Every language file has the same index shape (same ids) with a
  different pool (different text), so the id is language-independent and the pool is the only thing that
  changes.

That last point is what makes the label system work ([C30.1](01-label-system.md)): a string id means the same
*thing* in every language, resolved to different *text* by each language's pool.

> ✅ *Verified:* `Labels.bin` and `English.bin` are the same `0x00039000` chunk with identical header words
> (`0x10, 0x11A8, 0x1814, 0xA554, 0x100`) — one format, sub-table offsets + string pool.
> 🟡 *Reasoned:* the precise index encoding (hash vs sorted array) and string-pool framing (null-terminated vs
> length-prefixed) are the format's detail; the header-offsets + pool structure and the one-format fact are
> verified.

## Editing implications

- **Edit strings in the pool** — change text by editing the pooled string; if the length changes, the pool and
  the index offsets shift (a repack) ([C30.6](06-editing-text.md)).
- **Same-length edits are simplest** — replacing a string with one of equal byte length avoids re-stamping
  offsets, the string-table version of an in-place edit.
- **Keep the index consistent** — every id must resolve to a valid pool offset; adding a string means adding an
  index entry and a pool string.
- **Preserve the header offsets** — the sub-table offsets must match the (possibly resized) tables after an
  edit.

---

### Key takeaways

- All localization files share the chunk `0x00039000` format: a header of **offsets to sub-tables** plus a
  **string pool**.
- The structure is an **id-keyed index** (id → pool offset) + the **string pool** (the text).
- Index + pool gives fast lookup, compact ids, and one index shape across languages (id language-independent,
  pool per-language).
- Verified identical headers in `Labels.bin` and `English.bin` confirm one format.
- Edit strings in the pool (same-length is in-place; resizing repacks the index/offsets); keep the index and
  header consistent.

**Continue:** [C30.3 — Labels.bin](03-labels.md) · [Chapter 30 hub](C30-Localization-Labels.md)
