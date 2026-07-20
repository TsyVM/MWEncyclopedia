# C30.5 — Image Text & Fonts

> **The one-sentence version:** not all text is rendered from strings — where a word is art (a logo, a stylised
> graphic), it's an image in a per-language TPK (`LanguageTextures.bin`) localized by swapping the picture,
> while ordinary text is strings drawn with fonts.

[← C30.4 — The language files](04-language-files.md) · [Chapter 30 hub](C30-Localization-Labels.md) ·
[Next: C30.6 — Translating & adding text →](06-editing-text.md)

---

## Two kinds of localizable text

The UI has two ways to show words, and both are localizable:

| | Glyph text | Image text |
|---|---|---|
| Stored as | a string (id → text) | a picture (a TPK region) |
| Rendered by | fonts ([Ch 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) | drawn as an atlas image ([C27.2](../C27-FrontEnd-Shell-UI/02-ui-atlases.md)) |
| Localized by | swapping the language file ([C30.4](04-language-files.md)) | swapping the per-language image |
| Used for | ordinary UI text | logos, stylised words, baked graphics |

Most text is **glyph text** — the string system of this chapter. But where a word is *art* — a game logo, a
stylised heading, a word integrated into a graphic — it's **image text**, a picture rather than a string.

## Why some text is a picture

A word becomes an image when it needs to be more than glyphs can give:

- **Stylised typography.** A logo or heading with custom lettering, effects, or integration into artwork can't
  be produced by the font renderer ([C28.4](../C28-Fonts-Glyphs/04-rendering.md)) — it's authored as art.
- **Baked graphics.** Text that's part of a larger image (a sign, a banner) is just pixels in that image.
- **Scripts or effects the font can't do.** Where glyph rendering is impractical, a pre-rendered image
  sidesteps it ([C28.5](../C28-Fonts-Glyphs/05-codepoints.md)).

So image text is the escape hatch for words that are pictures — and, being pictures, they still need
localizing.

## Localized by swapping the image

Image text is localized the same way glyph text is — by having a **per-language version** — but the version is
an *image*, not a string. The `LanguageTextures.bin` is a **TPK** ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md))
of localized image text: for each localized graphic, a per-language picture, selected by the active language
([C30.1](01-label-system.md)). So a stylised "START" logo has an English image, a French image, a German image,
and the active language picks which to draw — exactly parallel to picking the string for glyph text.

> ✅ *Verified (archive):* localized image text lives in a per-language `LanguageTextures.bin` TPK — the image
> counterpart to the string tables; both are selected by the active language.
> 🟡 *Reasoned:* the exact per-language image selection mechanism is the UI's detail; the TPK-of-localized-images
> structure is consistent with the verified TPK format and the label system.

## The complete text picture

Putting glyph text and image text together, the game's localizable text is:

```
UI element
├── glyph text  → label → id → string (active language file) → fonts → pixels   (C30.1–C30.4, Ch 28)
└── image text  → localized image (LanguageTextures.bin, active language) → pixels
```

Both are **content selected by the active language**: the string system for words rendered from glyphs, the
image system for words that are art. A fully localized build ships both — every language's strings *and* every
language's image-text pictures.

## Editing implications

- **Edit glyph text via strings** ([C30.6](06-editing-text.md)) — the common case.
- **Edit image text via the localized atlas** ([C5.5](../C5-Textures-TPK/05-extract-replace.md)) — for logos
  and stylised words.
- **Localize both** — a new localized graphic needs a per-language image, like a new string needs a
  per-language string.
- **Know which you're editing** — a word that won't change via strings is probably image text (a picture), and
  vice versa.

---

### Key takeaways

- The UI has **two** localizable text kinds: **glyph text** (strings + fonts) and **image text** (pictures).
- Image text is used for logos, stylised words, and baked graphics that glyph rendering can't produce.
- It's localized by a **per-language image** in `LanguageTextures.bin` (a TPK), selected by the active language
  — parallel to picking a string.
- The full picture: glyph text (label→id→string→fonts) plus image text (localized picture), both language-
  selected.
- Edit glyph text via strings, image text via the localized atlas; localize both; identify which kind a word
  is.

**Continue:** [C30.6 — Translating & adding text](06-editing-text.md) · [Chapter 30 hub](C30-Localization-Labels.md)
