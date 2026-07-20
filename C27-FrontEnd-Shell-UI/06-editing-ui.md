# C27.6 — Re-skinning & Re-laying-out

> **The one-sentence version:** the UI edits along three independent axes — re-skin by editing atlas TPKs,
> re-lay-out by editing layout tables, re-word by editing strings — each without disturbing the others, as long
> as atlas regions, labels, and sizes stay consistent.

[← C27.5 — Text in the UI](05-ui-text.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md) ·
[Next: Chapter 28 — Fonts & Glyph Tables →](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)

---

## Three independent axes

The atlas + layout + label split ([C27.3](03-layout.md)) means UI editing decomposes cleanly:

| Goal | Edit | Chapter |
|---|---|---|
| Change how the UI **looks** | atlas TPKs | [C27.2](02-ui-atlases.md), [Ch 5–6](../C5-Textures-TPK/C5-Textures-TPK.md) |
| Change **where** things are | layout tables | [C27.3](03-layout.md) |
| Change **what text** says | strings/labels | [Ch 30](../C30-Localization-Labels/C30-Localization-Labels.md) |

Each axis edits independently — re-skin without moving anything, re-position without re-drawing art, re-word
without touching either. This is the payoff of the data-driven UI ([C27.1](01-shell-scenes.md)).

## Re-skinning (the safest edit)

Re-skinning changes the atlas art and nothing else:

- **Edit the atlas TPK** ([C5.5](../C5-Textures-TPK/05-extract-replace.md)) — swap button/panel/icon pixels.
- **Keep the same dimensions and format** — a same-size DXT/ARGB swap is an in-place TPK edit
  ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)); no layout or UV changes needed.
- **Keep image positions within the atlas** — if you move an image to a different spot in the atlas, the
  layout's UV rectangle for it must follow ([C27.2](02-ui-atlases.md)).

Same-size, same-position re-skins are the front-end equivalent of a same-size texture swap — the most reliable
UI mod.

## Re-laying-out

Moving or resizing elements edits the layout table ([C27.3](03-layout.md)):

- **Change an element's screen rectangle** to move/resize it — in the layout's coordinate convention
  (relative/anchored, [C27.3](03-layout.md)), so it still adapts to resolution.
- **Change its atlas region** to draw different art from the atlas.
- **Change its label** to show different text ([C27.5](05-ui-text.md)).
- **Add/remove elements** — insert or delete layout entries (a repack of the layout data).

The constraint is consistency: every element's UV rectangle must point at real atlas art, and every label must
exist in `Labels.bin`.

## Re-wording

Changing text is purely a string edit ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)):

- **Edit the string** in the language file to change what a label says.
- **Re-label an element** in the layout to show a different existing string.
- **For image text**, edit the localized atlas ([C27.5](05-ui-text.md)).

No scene or layout change is needed to re-word — the label indirection ([C27.5](05-ui-text.md)) handles it.

## The consistency rules

Across all three axes, keep the references valid:

- **UV regions → real atlas art.** An element's rectangle must land on an actual image in the atlas.
- **Labels → real strings.** An element's label must exist in `Labels.bin` and resolve to a string
  ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)).
- **Sizes/format on atlas edits.** Same-size, same-format for in-place; resize is a repack
  ([C5.5](../C5-Textures-TPK/05-extract-replace.md)).
- **Coordinate convention on layout edits.** Use the layout's anchoring so the UI stays correct across
  resolutions.

## Verify

After a UI edit, check the pieces the change touched:

1. **Atlas parses** and elements sample the right art ([C27.2](02-ui-atlases.md)).
2. **Layout references resolve** — every UV region and label is valid.
3. **Text resolves** — labels map to strings in the active language
   ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)).
4. **See it in game** — navigate the screens (and drive, for HUD) at your target resolution; elements are where
   and how you intended.

The in-game check is decisive for UI — layout and anchoring bugs (elements off-screen, overlapping, wrong
resolution) only show when rendered.

---

### Key takeaways

- UI edits along three independent axes: **re-skin** (atlas TPKs), **re-lay-out** (layout tables), **re-word**
  (strings/labels).
- Re-skinning same-size/same-format art is the safest edit — an in-place TPK swap.
- Re-laying-out edits screen rectangles, atlas regions, and labels in the layout's coordinate convention.
- Re-wording is a pure string/label edit; image text is a localized-atlas edit.
- Keep UV regions on real art, labels on real strings, and formats/sizes consistent; verify in game at your
  target resolution.

**Continue:** [Chapter 28 — Fonts & Glyph Tables](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md) ·
[Chapter 27 hub](C27-FrontEnd-Shell-UI.md)
