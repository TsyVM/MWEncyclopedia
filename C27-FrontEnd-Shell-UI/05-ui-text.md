# C27.5 — Text in the UI

> **The one-sentence version:** the shell never stores literal words — a scene references a label, the label
> system resolves it to the active language's string, and the fonts render it — so the UI is localizable
> without re-authoring a single screen.

[← C27.4 — The HUD](04-hud.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md) ·
[Next: C27.6 — Re-skinning & re-laying-out →](06-editing-ui.md)

---

## Labels, not literals

A UI element that shows text does **not** store the words — it stores a **label** (a stable name/ID). "Show
`MENU_START`" is what the scene contains; the actual string ("START", "COMMENCER", "STARTEN") comes from the
**label system** ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) at runtime:

```
UI scene → label MENU_START → Labels.bin: MENU_START = string-id → English.bin: string-id = "START"
                                                                    French.bin:  string-id = "COMMENCER"
```

The scene is **language-neutral**; the string it displays depends entirely on the active language file. This is
the mechanism that makes one set of screens serve every localization ([C30.1](../C30-Localization-Labels/01-label-system.md)).

## The three-stage lookup

Showing a piece of UI text is a three-stage resolution ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)):

1. **Scene → label.** The UI element names a label (`MENU_START`).
2. **Label → string id.** `Labels.bin` maps the label to a string id.
3. **String id → text.** The active language's `.bin` (e.g. `English.bin`) maps the id to the actual string.

Then the fonts ([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) render the string's glyphs into the
element's screen rectangle ([C27.3](03-layout.md)). So text flows: label → id → string → glyphs → pixels.

## Why the indirection

Two levels of indirection (scene→label, label→id, id→string) look like a lot, but each earns its place
([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)):

- **Localization.** Swapping the active language file changes every string at once, with no change to scenes or
  labels — the whole point.
- **Re-wording without re-authoring.** Change what `MENU_START` says by editing the string, not the screen.
- **Re-labelling without re-wording.** Point an element at a different label to change which text it shows,
  reusing existing strings.

It's the same "reference resolves to data" indirection as textures ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md))
and vault fields ([C12.5](../C12-Reflection-Schema/05-resolving-values.md)) — here applied to text.

## Some "text" is a picture

Not all UI words are rendered from strings. Where a word is **art** — a stylised logo, a baked graphic — it
lives in an atlas ([C27.2](02-ui-atlases.md)) as an image, and it's localized by having a per-language *image*
(the `LanguageTextures.bin` TPK, [C30.1](../C30-Localization-Labels/01-label-system.md)) rather than a string.
So the UI has two kinds of text: **glyph text** (rendered from strings via fonts) and **image text** (a
picture in an atlas). Both are localizable — one by string, the other by swapping the localized image.

> ✅ *Verified:* the language files (`English.bin` etc.) and `Labels.bin` are the string/label tables (EAGL
> chunk `.bin` files), and the label→id→string lookup is the UI text mechanism
> ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)).

## Editing implications

- **Change UI text by editing strings** ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) —
  no scene changes needed.
- **Change which text an element shows** by re-labelling it in the layout ([C27.6](06-editing-ui.md)).
- **For image text, edit the localized atlas** ([C27.2](02-ui-atlases.md), [C30](../C30-Localization-Labels/C30-Localization-Labels.md)) —
  it's a picture, not a string.
- **Keep labels valid** — an element referencing a label not in `Labels.bin` shows nothing (or a fallback).

---

### Key takeaways

- UI elements store a **label**, not literal words; the label system resolves it to the active language's
  string.
- The lookup is three stages: scene → label → string id → text, then fonts render the glyphs.
- The indirection enables localization, re-wording, and re-labelling — all independent of the screens.
- Some "text" is **image text** (a picture in an atlas), localized by swapping the per-language image.
- Edit UI text via strings (Chapter 30), which text via re-labelling, and image text via localized atlases.

**Continue:** [C27.6 — Re-skinning & re-laying-out](06-editing-ui.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md)
