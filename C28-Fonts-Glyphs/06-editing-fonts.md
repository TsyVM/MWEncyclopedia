# C28.6 — Editing Fonts

> **The one-sentence version:** re-skin a font freely by editing its atlas TPK (keeping glyphs in their
> rectangles, same format/size), but treat glyph-metric edits as read-only-ish because the glyph-table chunk id
> is unconfirmed and the record layout is heuristic.

[← C28.5 — Codepoints & localized scripts](05-codepoints.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md) ·
[Next: Chapter 29 — Minimap: Tiles & Map Data →](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)

---

## Two edits, two confidence levels

Font editing splits cleanly by which half you touch:

| Edit | Target | Confidence |
|---|---|---|
| **Re-skin** (restyle letters) | atlas TPK ([C28.3](03-atlas.md)) | ✅ fully supported |
| **Re-metric** (spacing/UVs) | glyph table ([C28.2](02-glyph-entry.md)) | 🟡 heuristic |

The atlas is a confirmed TPK, so re-skinning is as safe as any texture edit. The glyph table's chunk id is
**unconfirmed** ([C28.1](01-atlas-glyph-table.md)), so metric edits rely on a heuristic decode — reliable to
read, riskier to write.

## Re-skinning (the safe edit)

To restyle a font — a new typeface look, a stylised UI font — edit the **atlas**:

```python
def reskin_font(font, new_glyph_images):
    # redraw each letter within its existing UV rectangle on the atlas
    atlas = font.atlas
    for cp, img in new_glyph_images.items():
        rect = font.glyphs[cp]["uv"]           # keep the same rectangle (C28.2)
        draw_into_atlas(atlas, rect, img)      # new pixels, same location
    write_tpk_same_size(atlas)                 # in-place TPK swap (C5.5)
```

The rules are the texture-edit rules ([C5.5](../C5-Textures-TPK/05-extract-replace.md)):

- **Keep glyphs in their rectangles.** The glyph table's UVs still point where they did, so draw each new letter
  in the *same* atlas region ([C28.3](03-atlas.md)); the metrics are untouched.
- **Same format and dimensions.** A same-size DXT/ARGB swap is in-place ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md));
  resizing the atlas would require adjusting the (heuristic) UVs.
- **Preserve mips** ([C6.6](../C6-Texture-Codecs/06-mip-chains.md)) for clean scaling.
- **Recompress JDLZ** ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) after editing the bundle.

This re-skins the font's *look* without touching layout — new letters, same spacing.

## Re-metric (the cautious edit)

Changing spacing (advances) or glyph rectangles edits the **glyph table**, which is heuristic:

- **You *can* read metrics reliably** — advances and UVs decode ([C28.2](02-glyph-entry.md)) well enough to
  understand and even predict layout.
- **Writing them is riskier** — because the chunk id and exact field order aren't confirmed, a metric edit might
  land in the wrong field. Validate carefully (does text still lay out sensibly?) and prefer atlas edits when
  they achieve the goal.

So treat metric editing as *possible but unverified* — do it when you must (e.g. re-spacing a re-skinned font),
with extra validation, not as a routine edit.

## Adding glyphs (for new characters)

Supporting new characters ([C28.5](05-codepoints.md)) — e.g. a symbol or an additional script — means adding
*both*:

1. **A glyph image** on the atlas (in a free region), and
2. **A glyph table entry** (codepoint, its UV rectangle, an advance).

The atlas addition is straightforward; the table addition is the heuristic part ([C28.2](02-glyph-entry.md)),
and it grows the table (a repack). For most modding, re-skinning existing glyphs is far more common than adding
new ones.

## Verify

After a font edit:

1. **Atlas parses** and decodes ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).
2. **Glyph rectangles still align** — each character's UV region contains its (new) image
   ([C28.3](03-atlas.md)).
3. **Text renders** ([C28.4](04-rendering.md)) — draw sample strings and check letters, spacing, and alignment.
4. **Check the languages you support** — render text in each script the font covers
   ([C28.5](05-codepoints.md)).

The render-sample-strings check is decisive: a font is only right when real text draws correctly at the sizes
the UI uses.

---

### Key takeaways

- **Re-skinning** (atlas TPK) is fully supported; **re-metric** (glyph table) is heuristic — the chunk id is
  unconfirmed.
- Re-skin by redrawing letters in their existing UV rectangles, same format/dimensions/mips, recompress JDLZ.
- Read metrics freely; write them cautiously with validation; prefer atlas edits when possible.
- Adding new characters needs a glyph image *and* a (heuristic) table entry.
- Verify by parsing the atlas, checking rectangle alignment, and rendering sample strings in each supported
  script.

**Continue:** [Chapter 29 — Minimap: Tiles & Map Data](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md) ·
[Chapter 28 hub](C28-Fonts-Glyphs.md)
