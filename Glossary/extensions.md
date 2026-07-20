# Glossary — extension index & file identification

A file's extension is a hint, not a contract. Use this page to map an extension to its likely format
and the chapter that covers it, then **confirm by content** with the decision tree below — several
extensions in this game (`.BIN` most of all) are overloaded across completely unrelated formats.

## Extension → format → chapter

| Extension | Usual format | Chapter |
|---|---|---|
| `.BIN` | Overloaded: chunk tree (geometry/TPK), `VPAK` vault, or minimap tiles | 5 / 6 / 8 / 11 / 29 |
| `.BUN` | Chunk bundle (world, scenery, textures, geometry) | 8 / 15 / 16 / 36 |
| `.LZC` | JDLZ-compressed chunk stream (global / font data) | 3 / 28 |
| `.ABK` | Sound-effect bank (`ABKC`/`BNKl`, EA-XA) | 19 |
| `.SNR` + `.SPT` | Sound bank routing + payload | 19 |
| `.GIN` | Engine RPM-curve audio (EA-XAS) | 22 |
| `.MUS` | Music stream (`SCHl`/`SCDl`/`SCEl`) | 21 |
| `.MPF` | Music routing graph (PathFinder v5) | 21 |
| `.VP6` | EA Multimedia movie (VP6 + EA-ADPCM) | 23 |
| `.DDS` | DirectDraw Surface texture | 6 |
| `.LOC` | `LOCH`/`LOCI` memory-card / UI / text container | 31 |
| `.FX` | Shader / effect definition | 62 |
| `.ASM` | (in this data set) D3D9 shader-disassembly dumps | 62 |
| `.CSI` / `.CHI` | Event / cutscene script & data | 17 / 25 |
| `.ENG`, `.GER`, `.FRE`, … | Localized text per language | 30 |
| `attributes.bin`, `FE_ATTRIB.bin`, `gameplay.bin` | `VPAK` attribute vaults | 11 / 13 / 14 |
| `MINI_MAP*.BIN` | Minimap tile envelope | 29 |
| `.bak` / `.paintbak` / `.TESTBAK` | Backup copies (not original game data) | 75 |

## The "what is this file?" decision tree

Open the file and read the first bytes:

1. **First 4 bytes `'JDLZ'`?** → JDLZ-compressed. Decompress (C3), then **restart this tree** on the
   decompressed bytes — compression hides the real magic.
2. **First 4 bytes `'VPAK'`?** → attribute vault (C11).
3. **First 4 bytes `'DDS '`?** → standalone DDS texture (C6).
4. **First 4 bytes `'ABKC'` / `'BNKl'`?** → sound bank (C19). **`'SCHl'`?** → MUS stream (C21).
   **`'MPFF'`?** → music graph (C21). **`'LOCH'`?** → memory-card / localized container (C31).
5. **First 2 bytes `'MZ'`?** → Windows executable / DLL.
6. **Otherwise read the first 8 bytes as a chunk header** `{u32 id; u32 size;}`:
   - `id == 0xB3300000` → TPK texture pack (C5).
   - `id == 0x80134000` → solid geometry (C8).
   - `id` ∈ `{0x00034110, 0x00034112, 0x80034100, 0x0003414A, 0x0003B800}` → world / stream bundle (C15–C18).
   - first chunk `id == 0x0003A100` → minimap tiles (C29).
   - `id == 0x00E34009` present → cutscene animation (C24).
   - else it is still a chunk tree — **dump every `id`/`size`** and match against
     [chunk-ids.md](chunk-ids.md).
7. **None of the above, but mostly printable text?** → a text / script / config file; read it
   directly.

```python
# Minimal identifier — works on any file, no format-specific knowledge
import os, struct

def identify(path):
    with open(path, 'rb') as f:
        head = f.read(16)
    if head[:4] == b'JDLZ': return 'JDLZ-compressed (decompress, then re-identify)'
    if head[:4] == b'VPAK': return 'VPAK attribute vault'
    if head[:4] == b'DDS ': return 'DDS texture'
    if head[:4] in (b'ABKC', b'BNKl'): return 'sound bank'
    if head[:4] == b'SCHl': return 'MUS / EA audio stream'
    if head[:4] == b'MPFF': return 'music routing graph'
    if head[:4] == b'LOCH': return 'memory-card / localized container'
    if head[:2] == b'MZ':   return 'Windows executable/DLL'
    cid, csz = struct.unpack('<II', head[:8])
    known = {0xB3300000: 'TPK texture pack',
             0x80134000: 'solid geometry',
             0x0003A100: 'minimap tiles'}
    if cid in known: return known[cid]
    if (cid & 0x80000000) or csz < os.path.getsize(path):
        return f'EAGL chunk tree (first id 0x{cid:08X}, size {csz}) — dump and match chunk-ids.md'
    return 'unknown — inspect bytes'
```

Two subtleties that catch people:

- **`.BIN` is not one format.** A `.BIN` may be a chunk tree, a `VPAK` vault, or minimap tiles. Never
  decide by extension; run the tree. The single most common misidentification in the whole data set
  is reading a vault `.BIN` as a chunk tree (its first four bytes spell `VPAK`, not a chunk ID).
- **Compression is a wrapper, not a format.** A `.LZC` — or any file whose first bytes are `'JDLZ'` —
  tells you *nothing* about the content until you decompress it. Decompress first, identify second.

## Per-area file catalogues

The complete file-by-file listing of the entire game — every file, its type, size, what it does, and
its notable contents — lives in the catalogue pages, built out chapter-group by chapter-group:

`files-TRACKS` · `files-GLOBAL` · `files-CARS` · `files-SOUND` · `files-NIS` · `files-MOVIES` ·
`files-FRONTEND` · `files-LANGUAGES` · `files-CREDITS` · `files-MEMCARD` · `files-SUBTITLES` ·
`files-ROOT`.
