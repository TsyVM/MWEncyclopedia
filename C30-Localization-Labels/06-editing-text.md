# C30.6 — Translating & Adding Text

> **The one-sentence version:** translate by editing a language file's string pool (same-length is in-place,
> longer needs a repack), add text by adding a label to `Labels.bin` and a string per language, and always keep
> the label→id→string chain and the fonts consistent.

[← C30.5 — Image text & fonts](05-image-text.md) · [Chapter 30 hub](C30-Localization-Labels.md) ·
[Next: Chapter 31 — Save Data & Memory-Card Containers →](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)

---

## Translating (editing strings)

The common edit is changing what existing text says — retranslating or re-wording. It's a **string-pool edit**
in a language file ([C30.4](04-language-files.md), [C30.2](02-table-format.md)):

```python
def retranslate(lang_file, string_id, new_text):
    encoded = encode_for_script(new_text, lang_file.encoding)   # correct encoding (C30.4)
    if len(encoded) == lang_file.string_length(string_id):
        overwrite_in_place(lang_file, string_id, encoded)        # same length → in-place
    else:
        repack_pool(lang_file, string_id, encoded)               # different length → repack
```

Two rules:

- **Encode for the script.** Produce bytes in the language's encoding ([C30.4](04-language-files.md)) — Latin,
  UTF-16, etc. — or the text renders as garbage.
- **Same length is in-place; longer/shorter is a repack.** A same-byte-length replacement overwrites the pooled
  string without moving offsets ([C30.2](02-table-format.md)); a different length shifts the pool and the
  index — re-stamp both.

## Adding new text

Introducing a *new* piece of UI text touches three places ([C30.1](01-label-system.md)):

1. **`Labels.bin`** — add a label → id entry ([C30.3](03-labels.md)) so screens can reference it by name.
2. **Every language file** — add the string for that id, per language
   ([C30.4](04-language-files.md)) — the same id in each, with each language's text.
3. **The screen** — reference the new label where the text should appear
   ([C27.5](../C27-FrontEnd-Shell-UI/05-ui-text.md)).

Miss any and the chain breaks: no label → the screen can't reference it; no string in a language → that language
shows nothing; no screen reference → the text never appears.

## Keep the chain valid

Across all text edits, the label→id→string chain must hold ([C30.1](01-label-system.md)):

- **Every label has an id** in `Labels.bin`.
- **Every id has a string** in *every* language file (or a fallback).
- **Every screen label** exists in `Labels.bin`.

A break shows as missing or fallback text. The most common mistake when adding text is forgetting a language —
add the string to *all* language files, not just the one you're testing.

## Fonts and fit

Two rendering concerns ride along with text edits:

- **Fonts must have the glyphs.** New characters (especially CJK, [C28.5](../C28-Fonts-Glyphs/05-codepoints.md))
  need font glyphs, or they render as fallback/blank.
- **Text must fit its element.** A translated string can be much longer than the original
  ([C28.4](../C28-Fonts-Glyphs/04-rendering.md)); check it fits the UI element's rectangle
  ([C27.3](../C27-FrontEnd-Shell-UI/03-layout.md)) — resize the element, use a smaller font sheet, or shorten
  the text if it overflows.

Localization is famously where "it fit in English" breaks — German and Finnish strings run long, CJK needs
different fonts. Test text in every language you support.

## Image text

For text that's a picture ([C30.5](05-image-text.md)), editing is a **localized-atlas** edit
([C5.5](../C5-Textures-TPK/05-extract-replace.md)) — change the per-language image, not a string. A new
localized graphic needs a per-language image just as a new string needs a per-language string.

## Verify

After a text edit:

1. **The chain resolves** — label → id → string in the active language ([C30.1](01-label-system.md)).
2. **Encoding is correct** — the string reads back as the intended characters ([C30.4](04-language-files.md)).
3. **Fonts render it** — glyphs exist for the characters ([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)).
4. **In every language** — check the UI in each supported language: text appears, is correct, and fits.

The per-language in-game check is decisive — text bugs (missing strings, wrong encoding, overflow, missing
glyphs) show only when the UI renders each language.

---

### Key takeaways

- **Translate** by editing a language file's string pool, correctly encoded; same-length is in-place, else a
  repack.
- **Add text** by adding a label (`Labels.bin`) + a string in **every** language file + a screen reference.
- Keep the **label→id→string chain** valid — a break shows as missing/fallback text; don't forget any language.
- Fonts must have the glyphs, and translated text must **fit** its UI element (translations run long).
- Image text is a localized-atlas edit; verify the chain, encoding, fonts, and fit **in every language**.

**Continue:** [Chapter 31 — Save Data & Memory-Card Containers](../C31-Save-MemoryCard/C31-Save-MemoryCard.md) ·
[Chapter 30 hub](C30-Localization-Labels.md)
