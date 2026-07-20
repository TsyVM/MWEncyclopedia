# C28.5 — Codepoints & Localized Scripts

> **The one-sentence version:** because the game ships many languages — including Chinese and Korean — the
> fonts must carry the glyphs for each script's codepoints, so a language's font sheet covers exactly the
> characters its strings use.

[← C28.4 — Rendering text](04-rendering.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md) ·
[Next: C28.6 — Editing fonts →](06-editing-fonts.md)

---

## The game is multilingual

Most Wanted ships strings in many languages ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) —
the `LANGUAGES/` directory holds `English`, `French`, `German`, `Italian`, `Dutch`, `Danish`, `Finnish`,
`Chinese`, `Korean`, and more. Rendering those strings ([C28.4](04-rendering.md)) requires **glyphs for their
characters**, and different scripts need very different glyph sets:

- **Latin** (English, French, German, …) — ~100 letters, digits, punctuation, plus accented forms.
- **Cyrillic / Greek** — additional alphabets.
- **CJK** (Chinese, Korean) — thousands of ideographic glyphs.

The `codepoint` field ([C28.2](02-glyph-entry.md)) keys each glyph by character, so a font sheet is a set of
codepoint→glyph mappings covering some script.

## Fonts cover the codepoints they need

Because a CJK font needs thousands of glyphs and a Latin font a hundred, the fonts are **script-specific**: the
game uses font sheets that carry the codepoints of the active language's script. A bundle
([C28.1](01-atlas-glyph-table.md)) may hold multiple sheets, and the UI selects the one matching the text it's
drawing. So localization is a two-part system:

- **Strings** ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) — the words, per language.
- **Fonts** (this chapter) — the letterforms to render those words, per script.

Both must cover the language: the string table provides the characters, and the font must have glyphs for them
([C28.4](04-rendering.md)).

## CJK is the demanding case

Chinese and Korean stress the font system because of glyph count:

- **A Latin atlas** ([C28.3](03-atlas.md)) fits ~100 glyphs on a 256×256 or 512×256 sheet comfortably.
- **A CJK atlas** must hold thousands of ideographs, so it needs either a much larger atlas, multiple atlases,
  or a subset covering only the characters the game's strings actually use (common in localized console games —
  ship glyphs only for the text you display, not the whole language).

The UI text you saw earlier ([C27.5](../C27-FrontEnd-Shell-UI/05-ui-text.md)) is a bounded set of strings, so a
localized build ships the glyph subset those strings require — not an exhaustive font.

> ✅ *Verified:* the game ships many languages (`LANGUAGES/` includes `Chinese.bin`, `Korean.bin`, and European
> languages); the glyph `codepoint` field ([C28.2](02-glyph-entry.md)) keys glyphs by character.
> 🟡 *Reasoned:* the per-script font-sheet selection and CJK subsetting are the standard multilingual-font
> approach, consistent with the verified codepoint-keyed glyphs and the shipped languages.

## Image text sidesteps some glyphs

Where a word is **art** rather than rendered text ([C27.5](../C27-FrontEnd-Shell-UI/05-ui-text.md)) — a logo, a
stylised heading — it's an image in an atlas, localized by a per-language *image* (`LanguageTextures.bin`,
[C30](../C30-Localization-Labels/C30-Localization-Labels.md)), not a glyph string. This lets the game present
stylised or hard-to-render text without needing font glyphs for it — useful for logos and for scripts where a
baked image is simpler than glyph rendering.

## Editing implications

- **Match the font to the script.** Editing a language's text ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md))
  requires the font sheet to have glyphs for the characters used — adding characters may need atlas/table
  additions.
- **CJK edits are heavier** — adding ideographs means adding glyphs to a large atlas and table (metric editing
  is heuristic, [C28.2](02-glyph-entry.md)).
- **Subset awareness** — a localized build may only include glyphs for its shipped strings; new text needing new
  glyphs won't render without adding them.
- **Consider image text** for stylised or one-off words ([C27.5](../C27-FrontEnd-Shell-UI/05-ui-text.md)).

---

### Key takeaways

- The game is multilingual (`LANGUAGES/` incl. Chinese, Korean, European), so fonts must cover each script's
  codepoints.
- The glyph `codepoint` field keys glyphs by character; font sheets are **script-specific**, selected per text.
- Localization is two-part: **strings** (Chapter 30) provide characters, **fonts** provide letterforms — both
  must cover the language.
- **CJK** is the demanding case (thousands of glyphs) — large/multiple atlases or subsets of used characters.
- Match fonts to scripts when editing text; CJK edits are heavier; image text sidesteps some glyph needs.

**Continue:** [C28.6 — Editing fonts](06-editing-fonts.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md)
