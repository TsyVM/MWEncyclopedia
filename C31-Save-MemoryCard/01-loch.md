# C31.1 — The LOCH Container

> **The one-sentence version:** a `.loc` save opens with a `LOCH` header — magic, header size (`0x14`), version
> (`1`), item count, and total size — that wraps the `LOCI` items holding the save, a classic header/item
> container.

[← Chapter 31 hub](C31-Save-MemoryCard.md) · [Next: C31.2 — LOCI items →](02-loci.md)

---

## The header

Verified on `MEMCARD/LOCALE_ENGLISH.loc`, the `LOCH` header is:

```
+0x00  "LOCH"        magic (0x4C4F4348)
+0x04  u32  0x14     header size (20)
+0x08  u32  1        version
+0x0C  u32  1        item count
+0x10  u32  0x5C     total size / payload size
+0x14  "LOCI" …      the first item begins (C31.2)
```

`LOCH` = "LOC Header." It's a small, self-describing container header: the **magic** identifies the format, the
**version** pins the layout, the **count** says how many `LOCI` items follow, and the **size** bounds the file.
After the 20-byte header, the `LOCI` items begin.

## A header/item container

`LOCH`/`LOCI` is the same shape as containers throughout the engine — an outer header locating inner items:

| Container | Header | Items |
|---|---|---|
| Save | `LOCH` (this chapter) | `LOCI` records |
| Sound bank | `ABKC` ([C19.3](../C19-Audio-Banks/03-abk-bnkl.md)) | `BNKl` bank → `PT` |
| Event pack | `EventSequencePack` ([C25.1](../C25-NIS-Events/01-carp-scripts.md)) | `EventSequenceChunk` |
| TPK | `0xB3300000` ([C5.1](../C5-Textures-TPK/01-container-anatomy.md)) | Info + Data halves |

Recognising the pattern means you read `LOCH` the way you read any of these: parse the header, then walk the
`count` items using their sizes. The header's `count` and `size` are your cross-checks — the number of `LOCI`
items should match `count`, and their extents should fit within `size`.

## Reading the header

```python
def read_loch(buf):
    assert buf[:4] == b"LOCH"
    return {
        "header_size": u32(buf, 0x04),   # 0x14
        "version":     u32(buf, 0x08),   # 1
        "item_count":  u32(buf, 0x0C),   # 1
        "size":        u32(buf, 0x10),   # 0x5C
        "items_at":    u32(buf, 0x04),   # items begin after the header
    }
```

## Version matters

The `version` field (1) pins the container layout, so a robust reader checks it before trusting the offsets —
the same discipline as the TPK ([C5.2](../C5-Textures-TPK/02-metadata-tables.md)) and VPAK
([C11.1](../C11-Attribute-Vaults/01-vpak-header.md)) version checks. A different save version could relocate
fields or change the item format; MW ships version 1, and the check is cheap insurance against feeding a tool
the wrong save.

## Why wrap the save at all

The `LOCH` container serves the same purposes any save wrapper does:

- **Self-description.** The header says how big the save is and how many items it holds, so the loader reads it
  without external metadata.
- **Versioning.** The version field lets the game evolve the save format and still recognise old saves.
- **Structure for integrity.** A bounded container is where a checksum/size ([C31.4](04-integrity.md)) can be
  computed and verified.

So `LOCH` is the frame that makes the save loadable, versionable, and verifiable — the payload
([C31.3](03-save-payload.md)) lives inside it.

## Editing implications

- **Preserve the header on edits** — keep `magic`, `version`, and update `count`/`size` if items change
  ([C31.6](06-editing-saves.md)).
- **Check the version** before parsing — don't assume the layout on an unknown version.
- **`count` and `size` must stay truthful** — they bound the items; a wrong count/size fails the load or the
  integrity check ([C31.4](04-integrity.md)).
- **The container is small** — the bulk is the payload inside the items ([C31.2](02-loci.md)).

---

### Key takeaways

- A `.loc` save opens with a **`LOCH`** header: magic, header size (`0x14`), version (`1`), item count, total
  size.
- `LOCH`/`LOCI` is a **header/item container**, the same pattern as `ABKC`/`BNKl`, event pack/chunk, and TPK.
- Read the header, then walk `count` `LOCI` items by size; `count`/`size` are cross-checks.
- **Check the version** before trusting offsets — MW is version 1.
- The container gives self-description, versioning, and a frame for integrity; preserve it and keep `count`/
  `size` truthful on edits.

**Continue:** [C31.2 — LOCI items](02-loci.md) · [Chapter 31 hub](C31-Save-MemoryCard.md)
