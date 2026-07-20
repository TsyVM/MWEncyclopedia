# Chapter 28 — Fonts & Glyph Tables

> **Goal of this chapter:** decode how the game renders text — a font is a texture atlas of letterforms plus a
> glyph table that maps each character to a UV rectangle and an advance width — so you can re-skin the
> letterforms and understand text layout.

Every string the UI shows ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md),
[Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) is drawn with a **font**, and a font in
Most Wanted is the now-familiar **atlas + table** pattern applied to letters: an atlas texture holds the glyph
images, and a glyph table says where each character is on the atlas and how wide it sits. This chapter decodes
that structure.

> **Grounded in verified formats.** A font bundle is JDLZ-compressed ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md));
> decompressed, it is a chunk stream of interleaved **`TPK`, glyph-table** pairs — each glyph table pairs with
> the **preceding** TPK atlas ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)). A glyph entry is ≈20 bytes:
> `codepoint (u16)`, a float UV rectangle `(u0, v0, u1, v1)`, and a float `advance`. The atlas is a normal
> DXT1/DXT3 TPK (e.g. 256×256 / 512×256) — always exportable; the glyph-table chunk id is unconfirmed, so
> metric editing is heuristic.

---

## Deep-dive pages

- [C28.1 — The atlas + glyph table](01-atlas-glyph-table.md): the font's two halves and how they pair.
- [C28.2 — The glyph entry](02-glyph-entry.md): the ≈20-byte record — codepoint, UV rectangle, advance.
- [C28.3 — The atlas](03-atlas.md): the letterform texture, a normal TPK.
- [C28.4 — Rendering text](04-rendering.md): from a string to pixels — look up, sample, advance.
- [C28.5 — Codepoints & localized scripts](05-codepoints.md): covering the languages the game ships.
- [C28.6 — Editing fonts](06-editing-fonts.md): re-skinning letterforms vs the read-only metric caveat.

---

## 28.1 A font is atlas + glyph table

A font is two paired parts ([C28.1](01-atlas-glyph-table.md)):

- **The atlas** — a TPK texture ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) holding the rendered
  letterforms, packed into a sheet ([C28.3](03-atlas.md)).
- **The glyph table** — a small table mapping each character to its **UV rectangle** on the atlas and its
  **advance** width ([C28.2](02-glyph-entry.md)).

In the font bundle these interleave — `TPK, glyph-table, TPK, glyph-table, …` — so each glyph table belongs to
the TPK immediately before it. This is exactly the UI's atlas + table pattern
([C27.3](../C27-FrontEnd-Shell-UI/03-layout.md)): imagery in a texture, positioning in a table.

## 28.2 The glyph entry

Each glyph is a ≈20-byte record ([C28.2](02-glyph-entry.md)):

```
GlyphEntry:
  codepoint (u16)                 — the character
  u0, v0, u1, v1 (4 × float)      — its rectangle on the atlas
  advance (float)                 — how far the cursor moves after drawing it
```

The **codepoint** is the character's identity; the **UV rectangle** locates its image on the atlas; the
**advance** is its layout width. Three facts — which character, where its picture is, how wide it sits — are
all a renderer needs ([C28.4](04-rendering.md)).

## 28.3 The atlas is a normal TPK

The letterform sheet is an ordinary **TPK** ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) in DXT1/DXT3,
commonly 256×256 or 512×256 ([C28.3](03-atlas.md)). So the atlas is fully **exportable and re-skinnable** with
the texture tools ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) — you can restyle the font's *look*
by editing atlas pixels, exactly as you re-skin UI art ([C27.2](../C27-FrontEnd-Shell-UI/02-ui-atlases.md)).

## 28.4 Rendering a string

Drawing text is a per-character loop ([C28.4](04-rendering.md)):

```python
def draw_text(s, glyphs, atlas, x, y):
    for ch in s:
        g = glyphs[ord(ch)]                       # look up the glyph (C28.2)
        blit(atlas, g.uv_rect, x, y)              # sample its atlas rectangle, draw it
        x += g.advance                            # move the cursor by its advance
```

For each character: look up its glyph, sample its atlas rectangle, draw it, advance the cursor. The **atlas
holds what the letters look like**; the **glyph table holds where each letter is and how wide it sits** — the
same separation as every atlas + table system.

## 28.5 Localized scripts

Because the game ships many languages ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) —
including Chinese, Korean, and European scripts — the fonts must cover their **codepoints**. The `codepoint`
field ([C28.2](02-glyph-entry.md)) keys glyphs by character, and different-language fonts carry the glyphs
their scripts need ([C28.5](05-codepoints.md)). So the font system is the rendering half of localization: the
strings come from the language files, and the fonts provide the letterforms to draw them.

---

### Key takeaways

- A font is **atlas + glyph table**: a TPK of letterforms plus a table mapping characters to UV rectangles and
  advances.
- Font bundles are JDLZ, interleaving `TPK, glyph-table` — each table pairs with the preceding atlas.
- A glyph entry (≈20 B) is `codepoint (u16)`, a float UV rectangle, and a float `advance`.
- The atlas is a normal DXT TPK — always re-skinnable; the glyph-table chunk id is unconfirmed (metrics
  heuristic).
- Rendering is look-up → sample → advance; fonts cover the codepoints of every shipped language.

**Next:** [Chapter 29 — Minimap: Tiles & Map Data](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md): the same
atlas + table idea applied to the map.
