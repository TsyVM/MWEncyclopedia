# Glossary

The glossary is the reference desk of the encyclopedia. Where a chapter *teaches* a subsystem, the
glossary lets you *look something up* — an acronym you hit mid-chapter, a chunk ID you dumped out of
an unknown file, an extension you don't recognise, or "which file even holds this data?"

It has four conceptual pages and a file catalogue:

| Page | Use it when… |
|---|---|
| [terminology.md](terminology.md) | you meet a term — EAGL, chunk, container bit, Joaat, FVF, VPAK, vtable, singleton — and want the one-paragraph definition plus a pointer to the chapter that unpacks it. |
| [chunk-ids.md](chunk-ids.md) | you dumped a file and want to know what a `0x8xxxxxxx`/`0x0xxxxxxx` ID *means*, whether it's a container, and what its payload looks like. |
| [extensions.md](extensions.md) | you have a file and want to know what it probably is — by extension, then confirmed by content with the decision tree. |
| **File catalogue** | you know *what* you want to change (a specific car, a specific sound) and need *which file* holds it. Built out per chapter-group: `files-TRACKS`, `files-GLOBAL`, `files-CARS`, `files-SOUND`, `files-NIS`, `files-MOVIES`, `files-FRONTEND`, `files-LANGUAGES`, `files-CREDITS`, `files-MEMCARD`, `files-SUBTITLES`, `files-ROOT`. |

## The identification workflow, in one breath

You are handed a file. You want to know what it is and where in the book to go.

1. **Read the first 16 bytes.** A recognised FourCC (`JDLZ`, `VPAK`, `DDS `, `ABKC`/`BNKl`, `SCHl`,
   `MPFF`, `LOCH`, `MZ`) settles it immediately — jump to that format's chapter.
2. **`JDLZ`?** Decompress first (Chapter 3), then start over on the decompressed bytes — compression
   hides the real magic.
3. **No magic?** Read the first 8 bytes as a chunk header `{u32 id; u32 size;}` and look the ID up in
   [chunk-ids.md](chunk-ids.md). Bit 31 of the ID tells you container vs. leaf before you know
   anything else.
4. **Still unknown?** Dump the whole chunk tree and match the IDs you find. An EAGL file always
   yields a clean tree of known IDs; if it doesn't, it isn't a chunk tree.

That workflow is the connective tissue between the three conceptual pages, and it is the same one the
book's universal opener automates (see [C1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)).

## Conventions used throughout the glossary

- Numbers are hex with a `0x` prefix unless written with a decimal point or an explicit unit.
- "LE"/"BE" mean little-/big-endian. The game is LE except the EA audio stream fields (BE) — see
  [terminology → Endianness](terminology.md#numbers--identifiers).
- Confidence markers (✅ verified, 🟡 reasoned, ⏳ open) carry the same meaning as in the chapters;
  see the [main README](../README.md#confidence-markers).
