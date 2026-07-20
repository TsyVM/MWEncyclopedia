# C1.7 — The Non-Chunk Container Models

> **The one-sentence version:** a few important files — attribute vaults, EA audio, and DDS textures —
> are *not* chunk trees at all; recognise them by their magic at offset 0 and branch to a different
> parser before you ever try to read a chunk header.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.6 — Matrices & coordinates](06-matrices-and-coordinates.md) ·
[Next: C1.8 — The compression boundary →](08-compression-boundary.md)

---

## What it is

The chunk tree is the *dominant* container model, not the *only* one. Three other models ship inside the
game, each announced by a FourCC or magic at file offset 0. If you feed one of these to a chunk walker,
the walker will read the magic bytes as an `{id, size}` pair and immediately wander off — the first
eight bytes of `VPAK…` are not a chunk header, so the "size" is nonsense and the walk desyncs on byte
one.

| Magic (offset 0) | Model | Why it's different | Chapter |
|---|---|---|---|
| `'VPAK'` | Attribute vault | Tagged blocks (`ErtS`/`NpeD`/`NrtS`/`NtaD`) + a string table + typed records; a schema-driven database, not a chunk tree | [C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md) |
| `'ABKC'` / `'BNKl'` | Sound bank | EA audio: TLV records with **big-endian** fields | [C19](../C19-Audio-Banks/C19-Audio-Banks.md) |
| `'SCHl'` | Music / EA stream | Block-tagged EA stream (`SCHl`/`SCCl`/`SCDl`/`SCEl`), big-endian fields | [C21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md) |
| `'MPFF'` | Music routing graph | PathFinder graph, its own record layout | [C21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md) |
| `'DDS '` | Texture | Fixed 128-byte header + pixel data; no chunks | [C6](../C6-Texture-Codecs/C6-Texture-Codecs.md) |
| `'LOCH'` | Memory-card / localized container | `LOCH`/`LOCI` records | [C31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md) |
| `'MZ'` | Windows executable / DLL | PE format | — |

## How to branch

The rule is: **magic first, chunk header last.** Read the first four bytes (two for `MZ`) and compare
against the known FourCCs. Only if none match do you fall through to reading an `{id, size}` chunk
header. This is exactly the ordering the book's universal opener uses
([C1.9](09-universal-opener.md)):

```python
def container_kind(head16):
    m4 = head16[:4]
    if m4 == b'JDLZ': return 'compressed'          # decompress, then re-identify (C1.8)
    if m4 == b'VPAK': return 'vault'               # C11
    if m4 == b'DDS ': return 'dds'                 # C6
    if m4 in (b'ABKC', b'BNKl'): return 'bank'     # C19
    if m4 == b'SCHl': return 'mus'                 # C21
    if m4 == b'MPFF': return 'music_graph'         # C21
    if m4 == b'LOCH': return 'loc'                 # C31
    if head16[:2] == b'MZ': return 'pe'
    return 'chunks'                                # the common case
```

Note that a `.BIN` extension tells you nothing here — a vault, a chunk tree, and minimap tiles all use
`.BIN`. The first four bytes are what settle it. The most common misidentification in the entire data
set is reading a vault `.BIN` as a chunk tree; the `VPAK` magic is the thing that prevents it.

## Why it is designed this way

These formats predate or sit orthogonally to the EAGL chunk model, and each has a shape the chunk model
would fit poorly:

- **A vault is a database.** It needs a string table, a symbol table, and typed records that reference
  each other by name-hash. A chunk tree could *hold* that, but the vault's own tagged-block layout is
  purpose-built for schema-driven records with `default` inheritance — a very different access pattern
  from "walk and dispatch." See [C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md).
- **EA audio is a cross-platform stream format.** It carries its own big-endian TLV records precisely so
  the same audio bytes work on big-endian consoles ([C1.5](05-endianness-islands.md)). It was never an
  EAGL construct; the engine just embeds it.
- **DDS is a Microsoft interchange format.** It is the standard container D3D itself understands, so the
  game stores loose textures in it verbatim rather than re-wrapping them in chunks.

In short, the chunk model is EA Black Box's *engine* container; these three are *foreign* containers the
engine consumes. Keeping them in their native form was cheaper and more compatible than converting
everything to chunks.

## Bending it — respect the boundary

- **Never run a chunk walker on a non-chunk file.** It will produce plausible-looking garbage (random
  ids, random sizes) that wastes hours. Branch on the magic first, every time.
- **Don't "chunk-wrap" a vault or a DDS to make it fit your tool.** The game expects these at their
  native offset-0 magic. Wrapping one in a chunk header makes it unreadable to the engine even though
  your tool is happy.
- **Do treat compression as a pre-layer.** `JDLZ` can wrap *any* of these — a compressed vault is `JDLZ`
  at offset 0 and `VPAK` only after decompression. Always decompress first, then re-run the branch. That
  ordering is the subject of the next page.

---

**Continue:** [C1.8 — The compression boundary](08-compression-boundary.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
