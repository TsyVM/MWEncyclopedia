# C1.9 — Building a Universal Opener & Dumper

> **The one-sentence version:** one small, hardened function that decompresses, identifies, walks, and
> prints any file in the game — the tool you will paste into the front of every experiment for the rest
> of the book.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.8 — Compression boundary](08-compression-boundary.md) ·
[Next: C1.10 — Editing & repacking →](10-editing-and-repacking.md)

---

## What it is

Everything in this chapter so far — the container bit, the size stride, the magic branch, the
compression boundary — composes into a single reusable tool. It takes a path and returns a structured
view: either a decoded non-chunk container tag, or the full chunk tree. Layered on top is a **dumper**
that prints the tree so you can eyeball any file in seconds.

This is not throwaway code. Every later chapter opens with "dump the file and find chunk X"; this is the
`dump` they mean.

## The opener

```python
import os, struct

# from Chapter 3; stubbed here so the opener is runnable before you read C3
def jdlz_decompress(buf):
    raise NotImplementedError("see Chapter 3")

def walk_tree(buf, base=0, depth=0):
    off = 0
    while off + 8 <= len(buf):
        cid, size = struct.unpack_from('<II', buf, off)
        if off + 8 + size > len(buf):
            break
        payload = memoryview(buf)[off + 8: off + 8 + size]
        yield depth, base + off, cid, size, payload
        if cid & 0x80000000:
            yield from walk_tree(payload, base + off + 8, depth + 1)
        off += 8 + size

MAGICS = {
    b'VPAK': 'vault',   b'DDS ': 'dds',   b'ABKC': 'bank', b'BNKl': 'bank',
    b'SCHl': 'mus',     b'MPFF': 'music_graph', b'LOCH': 'loc',
}

def open_eagl(path):
    """Return (kind, payload). kind is 'chunks' | one of MAGICS' values | 'pe' | 'unknown'."""
    buf = open(path, 'rb').read()
    # 1) compression boundary — must come first
    if buf[:4] == b'JDLZ':
        buf = jdlz_decompress(buf)
    # 2) non-chunk container magics
    if buf[:4] in MAGICS:
        return MAGICS[buf[:4]], buf
    if buf[:2] == b'MZ':
        return 'pe', buf
    # 3) chunk tree — validate the first header looks sane before committing
    if len(buf) >= 8:
        cid, size = struct.unpack_from('<II', buf, 0)
        if (cid & 0x80000000) or (8 + size <= len(buf)):
            return 'chunks', list(walk_tree(buf))
    return 'unknown', buf
```

The one non-obvious line is the first-header sanity check in step 3. A real chunk file's first chunk is
either a container (top bit set) or a leaf whose `8 + size` fits inside the file. If neither holds, the
bytes are not a chunk tree and you return `unknown` rather than emitting a bogus walk — this is what
keeps the opener from confidently mis-parsing a format it has never seen.

## The dumper

```python
# name lookups keep the dump readable — extend from Glossary/chunk-ids.md
CHUNK_NAMES = {
    0x80134000: 'GeometryContainer', 0x80134010: 'GeometryObject',
    0x00134011: 'GeometryObjectHeader', 0x80134100: 'GeometryMesh',
    0xB3300000: 'TPKContainer', 0x0003B800: 'WorldMapData(CARP)',
    # …
}

def dump(path, max_payload_preview=0):
    kind, data = open_eagl(path)
    if kind != 'chunks':
        print(f"{os.path.basename(path)}: non-chunk container = {kind!r}")
        return
    for depth, off, cid, size, payload in data:
        name = CHUNK_NAMES.get(cid, '')
        flag = 'C' if (cid & 0x80000000) else ' '
        line = f"{'  '*depth}[{flag}] 0x{cid:08X} {name:<22} size={size:<8} @0x{off:X}"
        if max_payload_preview and not (cid & 0x80000000):
            preview = bytes(payload[:max_payload_preview]).hex(' ')
            line += f"  {preview}"
        print(line)
```

Example output on a solid-geometry bundle:

```
[C] 0x80134000 GeometryContainer      size=51284    @0x0
  [C] 0x80134001 GeometryInfo         size=132      @0x8
    [ ] 0x00134002 GeometryInfoHeader size=52       @0x10
    [ ] 0x00134003 GeometryHashTable  size=64       @0x4C
  [C] 0x80134010 GeometryObject       size=4176     @0x94
    [ ] 0x00134011 GeometryObjectHeader size=160    @0x9C
    [C] 0x80134100 GeometryMesh       size=3992     @0x144
      [ ] 0x00134900 MeshHeader       size=48       @0x14C
      [ ] 0x00134B02 MeshShadingGroups size=208     @0x184
      [ ] 0x00134B01 MeshVertices     size=2880     @0x25C
      [ ] 0x00134B03 MeshIndices      size=768      @0xDA0
```

Read top to bottom that is the entire structure of the file: a solid list holding one object, whose mesh
holds a header, its shading groups, its vertices, and its indices. You have not written a line of
geometry-specific code and you already know exactly where the vertex buffer lives (`@0x25C`, 2880 bytes)
— which is all you need to start [Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md).

## Why it is designed this way

The opener/dumper mirrors the engine's own load path deliberately (see [C1.12](12-runtime-view.md)): it
decompresses, branches on container model, and walks-and-dispatches. Because it runs the same algorithm
the game does, "the dumper parses it cleanly" is strong evidence "the game will load it," and vice
versa. That equivalence turns the dumper into a validator, not just a viewer — a use it gets in
[C1.11](11-failure-modes.md).

## Bending it — how to grow the tool without breaking it

- **Keep `open_eagl` pure — no printing, no editing.** Return data; let callers decide what to do. This
  is what lets you reuse it as the front end of an extractor, a patcher, and a diff tool without
  copy-pasting the parse.
- **Grow `CHUNK_NAMES` from the glossary, not from guesses.** An unnamed id in a dump is a signal to go
  identify it, not to invent a label.
- **Add a `--raw` mode that dumps unknown ids with a payload preview.** When you meet a file the opener
  calls `unknown`, a hex preview of the first bytes is usually enough to spot a magic you hadn't listed.
- **Resist special-casing.** The moment your opener has an `if filename == 'something.bin'` branch, it
  has stopped being universal. The whole point is that the *bytes* drive the parse.

---

**Continue:** [C1.10 — The size tree in practice: editing & repacking](10-editing-and-repacking.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
