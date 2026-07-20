# C31.2 — LOCI Items

> **The one-sentence version:** inside the `LOCH` container are `LOCI` records — each a tagged item with its own
> header and payload — that hold the pieces of the save, counted and located by the `LOCH` header.

[← C31.1 — The LOCH container](01-loch.md) · [Chapter 31 hub](C31-Save-MemoryCard.md) ·
[Next: C31.3 — The save payload →](03-save-payload.md)

---

## The items

Following the `LOCH` header ([C31.1](01-loch.md)) are its **`LOCI`** items — verified, the first at `+0x14`:

```
+0x14  "LOCI"        item tag (0x4C4F4349)
+0x18  u32  0x48     item size (72)
+0x1C  u32  0x0E     (item field — type / sub-count)
…      LOCIH …       item sub-header, then item data
```

`LOCI` = "LOC Item." Each is a self-contained record with a **tag**, a **size**, and a **payload**; the `LOCH`
`count` says how many there are ([C31.1](01-loch.md)). The verified file has one `LOCI` of 72 bytes, carrying a
`LOCIH` sub-marker (an item header) before its data. So the nesting is `LOCH` → `LOCI` → (`LOCIH` header +
data).

## Walking the items

```python
def walk_loci(buf, loch):
    items, off = [], loch["items_at"]
    for _ in range(loch["item_count"]):
        assert buf[off:off+4] == b"LOCI"
        size = u32(buf, off + 4)
        items.append({"off": off, "size": size, "data": buf[off+8 : off+8+size]})
        off += 8 + size                      # advance to the next item
    return items
```

Walk `count` items, each located by its size — the same size-advancing walk as any tagged-record container
([C19.3](../C19-Audio-Banks/03-abk-bnkl.md)). The items partition the container's payload; their sizes should
sum (with headers) to fit the `LOCH` size ([C31.1](01-loch.md)) — the save's version of "the parts fill the
whole."

## What items hold

Each `LOCI` holds a **piece of the save** ([C31.3](03-save-payload.md)) — the container splits the save into
items so different parts (career, settings, per-slot data) can be structured, located, and validated
separately. A save might have items for the career payload, the options, and metadata; the `LOCH` count and the
item tags/fields distinguish them. The `LOCIH` sub-header inside each item likely carries the item's own
type/version/size, nesting the self-description one level deeper.

> ✅ *Verified:* the `LOCH` container holds `LOCI` items (`LOCI` tag at `+0x14`, size `0x48`), with a `LOCIH`
> sub-marker — a nested header/item structure.
> 🟡 *Reasoned:* the exact `LOCI`/`LOCIH` field meanings (type, version, sub-count) are identified by role;
> the `LOCH`→`LOCI`→data nesting and the item tag/size are verified.

## Why split the save into items

Structuring the save as items rather than one blob buys the usual container benefits:

- **Separable pieces.** Career, settings, and metadata can be read/written independently.
- **Located by size.** Each item is found by walking sizes, so the loader reaches any piece directly.
- **Per-item integrity.** Each item can carry its own size/checksum ([C31.4](04-integrity.md)), so corruption
  can be localised.
- **Extensible.** New save data can be a new item without disturbing existing ones.

It's the same reasoning as every other container in the book — self-describing items in a counted container —
applied to a save.

## Editing implications

- **Edit item payloads, keep sizes truthful** — change data within a `LOCI`; if the size changes, update the
  item size and the `LOCH` size/count ([C31.1](01-loch.md)).
- **Preserve tags and sub-headers** — the `LOCI`/`LOCIH` framing must stay intact for the loader to find items.
- **Respect the count** — the number of `LOCI` items must match the `LOCH` `count`.
- **Recompute integrity** after any item edit ([C31.4](04-integrity.md)).

---

### Key takeaways

- Inside `LOCH` are **`LOCI`** items — tagged records (tag, size, payload) holding pieces of the save.
- Walk `count` items by size; they partition the container's payload (parts fill the whole).
- Each item holds a save piece; a `LOCIH` sub-header nests the self-description deeper (item type/version/size).
- Splitting the save into items gives separable, locatable, per-item-verifiable, extensible storage.
- Edit item payloads keeping sizes/count truthful, preserve the tags/sub-headers, and recompute integrity.

**Continue:** [C31.3 — The save payload](03-save-payload.md) · [Chapter 31 hub](C31-Save-MemoryCard.md)
