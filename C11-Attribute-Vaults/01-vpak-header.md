# C11.1 — The VPAK Header

> **The one-sentence version:** `attributes.bin` opens with the ASCII magic `VPAK`, a version (1), a block
> count (3), and an offset/size table that locates the string block, the data, and the trailer — a fixed
> purpose-built header, not an EAGL chunk tree.

[← Chapter 11 hub](C11-Attribute-Vaults.md) · [Next: C11.2 — The ErtS string table →](02-erts-strings.md)

---

## The header bytes

The first 64 bytes of the real `GLOBAL/attributes.bin`:

```
+0x00  "VPAK"          magic (0x4B415056 little-endian)
+0x04  u32  1          version
+0x08  u32  0x40       section-table / header size (64)
+0x0C  u32  3          block count
+0x10  u32  0
+0x14  u32  0x00055B7F offset/size (trailer-region related)
+0x18  u32  0x00052A10 offset/size
+0x1C  u32  0x00000080 → offset of the first block (ErtS, at 0x80)
+0x20  u32  0x00055C00 → offset of the trailer blocks (NpeD/NrtS/NtaD)
+0x24… zeros
```

Two fields orient you immediately: the block at `0x80` (the `ErtS` string table,
[C11.2](02-erts-strings.md)) and the trailer region at `0x55C00` (`NpeD`/`NrtS`/`NtaD`,
[C11.5](05-trailer-blocks.md)). Between them lies the reflection type-name table
([C11.3](03-type-names.md)) and the bulk of the file, the typed data records
([C11.4](04-data-records.md)).

## VPAK is not EAGL

It is worth stating plainly: a VPAK is **not** the `{u32 id, u32 size}` chunk tree of the texture and
geometry files ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)). It is a distinct
container with:

- a **fixed ASCII-magic header** (`VPAK`) rather than a numeric chunk id;
- **named blocks** identified by four-character tags (`ErtS`, `NpeD`, `NrtS`, `NtaD`) rather than numeric ids;
- **explicit offsets** in the header rather than a recursive size tree.

So the tree-walking tools from Chapter 1 do not apply here; VPAK gets its own reader. The payoff is that once
you know the header's offsets, you jump straight to each section — there is no walk to desynchronise.

## Reading the header

```python
def parse_vpak_header(buf):
    assert buf[:4] == b"VPAK", "not a VPAK file"
    version     = u32(buf, 0x04)
    header_size = u32(buf, 0x08)      # 0x40
    block_count = u32(buf, 0x0C)      # 3
    first_block = u32(buf, 0x1C)      # 0x80 → ErtS
    trailer     = u32(buf, 0x20)      # 0x55C00 → NpeD/NrtS/NtaD
    return {
        "version": version, "blocks": block_count,
        "erts_off": first_block, "trailer_off": trailer,
    }
```

## Why a fixed header suits the vault

The vault is loaded once, early, and queried constantly at runtime — the game asks "what is field *F* of
collection *C*?" thousands of times a frame. A fixed header with direct offsets means the loader can map the
string table, the type table, and the data region in one pass and build its lookup structures without walking
a tree. The design trades the EAGL format's uniform recursiveness for **fast, flat, offset-based access** —
appropriate for a database that is read far more than it is streamed.

## Version and portability

The version field (1) matters if you write tools: it pins the layout this chapter documents. A different
version could relocate fields, so a robust reader checks `version == 1` before trusting the offsets above,
exactly as a TPK reader checks its own version ([C5.2](../C5-Textures-TPK/02-metadata-tables.md)). Most Wanted
ships version 1 throughout; the check is cheap insurance against feeding a tool the wrong file.

---

### Key takeaways

- VPAK header: `VPAK` magic, version (1), block count (3), and offsets to the first block (`0x80`) and trailer
  (`0x55C00`).
- It is a **fixed, offset-based** container with four-char block tags — not an EAGL `{id, size}` chunk tree.
- Chapter 1's tree-walkers don't apply; VPAK gets a dedicated reader keyed on header offsets.
- The design favours fast flat access for a constantly-queried database.
- Check `version == 1` before trusting the documented offsets.

**Continue:** [C11.2 — The ErtS string table](02-erts-strings.md) · [Chapter 11 hub](C11-Attribute-Vaults.md)
