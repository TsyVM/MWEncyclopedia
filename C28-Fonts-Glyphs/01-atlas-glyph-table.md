# C28.1 — The Atlas + Glyph Table

> **The one-sentence version:** a font is a TPK atlas of letterforms paired with a glyph table, interleaved in
> the bundle as `TPK, glyph-table, TPK, glyph-table…`, so each table belongs to the atlas immediately before
> it.

[← Chapter 28 hub](C28-Fonts-Glyphs.md) · [Next: C28.2 — The glyph entry →](02-glyph-entry.md)

---

## Two halves, interleaved

A font bundle is JDLZ-compressed ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)); decompress it and
you get a standard chunk stream that alternates:

```
TPK atlas #1 · glyph table #1 · TPK atlas #2 · glyph table #2 · …
```

The interleaving is the key structural fact: **each glyph table associates with the *preceding* TPK**. A font
sheet is therefore a `{atlas, table}` pair, and a bundle may hold several (different sizes/styles/scripts). To
parse a font, walk the chunk stream and pair each glyph-table chunk with the TPK before it.

```python
def read_fonts(decompressed):
    chunks = walk_chunks(decompressed)
    fonts, pending_tpk = [], None
    for c in chunks:
        if is_tpk(c): pending_tpk = c
        elif is_glyph_table(c) and pending_tpk:
            fonts.append({"atlas": pending_tpk, "glyphs": parse_glyphs(c)})   # C28.2
    return fonts
```

## The atlas + table pattern, once more

This is the same idiom as the UI ([C27.3](../C27-FrontEnd-Shell-UI/03-layout.md)) and the minimap
([Chapter 29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)): **imagery in a texture, positioning in a
table.** For a font:

- **The atlas** holds *what the letters look like* — the rendered glyph pixels ([C28.3](03-atlas.md)).
- **The glyph table** holds *where each letter is and how wide it sits* — UV rectangle + advance
  ([C28.2](02-glyph-entry.md)).

Separating letterform *imagery* from letter *metrics* is what lets you re-skin the font's look (edit the atlas)
without disturbing its layout, and vice versa ([C28.6](06-editing-fonts.md)).

## Why interleave

Interleaving each atlas with its table (rather than grouping all atlases then all tables) keeps a font sheet
**self-contained and adjacent**: the loader reads an atlas and immediately the table that indexes it, binding
them without a separate lookup. It's a small locality optimisation, and it's why the pairing rule is
"preceding TPK" — the table follows the atlas it describes.

> ✅ *Verified (archive):* font bundles are JDLZ, decompressing to interleaved `TPK, glyph-table` pairs where
> each table pairs with the preceding TPK; the atlas is a normal DXT1/DXT3 TPK.
> 🟡 *Reasoned:* the glyph-table **chunk id** is unconfirmed — the table is identified heuristically (a payload
> dividing evenly into ≈20-byte glyph records); the atlas TPKs are unambiguous.

## Multiple sheets per font system

A bundle typically holds several `{atlas, table}` pairs because the game needs more than one font:

- **Different sizes** — a small font for dense text, a large one for headings.
- **Different styles** — a UI font, a stylised heading font.
- **Different scripts** — sheets covering the codepoints of different languages
  ([C28.5](05-codepoints.md)).

Each is its own atlas + table; the UI picks the sheet appropriate to the text it's drawing.

## Editing implications

- **Re-skin via the atlas TPK** — the reliable font edit ([C28.3](03-atlas.md), [C28.6](06-editing-fonts.md)).
- **Pair correctly** — when editing, keep each table with its preceding atlas; don't reorder the interleave.
- **Metrics are heuristic** — because the table chunk id is unconfirmed, treat glyph-metric edits cautiously
  ([C28.2](02-glyph-entry.md)).
- **Preserve JDLZ** — recompress the bundle after editing ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)).

---

### Key takeaways

- A font bundle is JDLZ, decompressing to interleaved `TPK, glyph-table` pairs; each table pairs with the
  **preceding** TPK.
- It's the **atlas + table** idiom: atlas = letterform imagery, glyph table = metrics (UV + advance).
- Interleaving keeps each font sheet self-contained and adjacent for the loader.
- A bundle holds multiple sheets (sizes, styles, scripts); the UI picks the right one per string.
- Re-skin via the atlas; keep pairs in order; treat metrics as heuristic; preserve JDLZ on rebuild.

**Continue:** [C28.2 — The glyph entry](02-glyph-entry.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md)
