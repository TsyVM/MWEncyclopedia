# C28.4 — Rendering Text

> **The one-sentence version:** drawing a string is a per-character loop — look up each glyph, sample its atlas
> rectangle into the screen, and move the cursor by its advance — so the atlas provides the letters' images and
> the glyph table provides their placement.

[← C28.3 — The atlas](03-atlas.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md) ·
[Next: C28.5 — Codepoints & localized scripts →](05-codepoints.md)

---

## The render loop

Rendering a string with a font is the atlas + table pattern in motion:

```python
def draw_text(s, font, x, y):
    for ch in s:
        g = font.glyphs.get(ord(ch))              # look up by codepoint (C28.2)
        if g is None:
            g = font.glyphs.get(FALLBACK)          # missing-glyph fallback
        blit(font.atlas, g["uv"], x, y)           # sample the atlas rectangle (C28.3)
        x += g["advance"]                          # advance the cursor (C28.2)
```

For each character: **look up** its glyph by codepoint, **sample** its UV rectangle from the atlas, **draw** it
at the cursor, and **advance** the cursor by the glyph's advance. That's the whole of single-line text
rendering — three steps per character, driven by the two font halves.

## Atlas provides imagery, table provides placement

The division of labour is the recurring one:

- **The atlas** ([C28.3](03-atlas.md)) answers *what does this letter look like* — its pixels.
- **The glyph table** ([C28.2](02-glyph-entry.md)) answers *where is this letter on the atlas* (UV rectangle)
  and *how wide does it lay out* (advance).

Neither alone renders text; together they do. This is exactly the UI's atlas + layout split
([C27.3](../C27-FrontEnd-Shell-UI/03-layout.md)) — pixels in a texture, placement in a table — applied at the
character level.

## Proportional layout via advance

Because each glyph has its own **advance** ([C28.2](02-glyph-entry.md)), text is **proportional**: `W` takes
more horizontal space than `i`, a space advances without drawing, and the cursor tracks the natural width of
what's been drawn. This is what makes the game's text look typeset rather than typewritten. Summing advances
gives a string's pixel width — used to center or right-align text in a UI element's rectangle
([C27.3](../C27-FrontEnd-Shell-UI/03-layout.md)):

```python
def text_width(s, font):
    return sum(font.glyphs[ord(c)]["advance"] for c in s)   # for alignment
```

## Fitting text into UI elements

The renderer draws into a UI element's screen rectangle ([C27.3](../C27-FrontEnd-Shell-UI/03-layout.md)), so
text layout interacts with the layout system:

- **Alignment** — center/right-align by computing `text_width` and offsetting the start `x`.
- **Wrapping** — if a string exceeds the element's width, break it into lines (advancing `y` by a line height).
- **Fitting** — very long localized strings ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md))
  may need a smaller font sheet ([C28.1](01-atlas-glyph-table.md)) or truncation to fit — a real localization
  concern, since a phrase in one language can be far longer in another.

So the font renders characters; the UI layout decides the box; together they place a string on screen.

> 🟡 *Reasoned:* the render loop and alignment/wrapping are the standard atlas-font rendering, consistent with
> the verified glyph fields (codepoint/UV/advance) and the UI layout system; the exact renderer code is the
> engine's.

## Editing implications

- **Re-skinning changes appearance, not layout** — new glyph pixels in the same rectangles draw differently but
  advance the same ([C28.3](03-atlas.md)).
- **Changing advances re-spaces text** — a metric edit ([C28.2](02-glyph-entry.md)) affecting alignment/width
  (heuristic, so cautious).
- **Watch localized string lengths** — a re-worded or translated string
  ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) may overflow its element; check fit.
- **Provide a fallback glyph** — for codepoints a sheet lacks, so missing characters degrade gracefully.

---

### Key takeaways

- Text rendering is a per-character loop: **look up → sample → advance**, driven by the atlas and glyph table.
- The atlas provides letter **imagery**; the glyph table provides **placement** (UV rectangle) and **width**
  (advance).
- Per-glyph advance gives **proportional** text; summing advances gives string width for alignment.
- Text draws into UI element rectangles — alignment, wrapping, and fitting connect fonts to the UI layout.
- Re-skins change look not layout; advance edits re-space text; watch localized string overflow and provide a
  fallback glyph.

**Continue:** [C28.5 — Codepoints & localized scripts](05-codepoints.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md)
