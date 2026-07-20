# C27.3 — Layout: Positioning the UI

> **The one-sentence version:** the front-end's recurring pattern is atlas + layout table — the atlas holds the
> pixels, and a table places each element on screen and maps it to its atlas region — so layout and art change
> independently.

[← C27.2 — UI atlases](02-ui-atlases.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md) ·
[Next: C27.4 — The HUD →](04-hud.md)

---

## Atlas + table, again

The single pattern behind the whole UI is **atlas + layout table**:

- **The atlas** ([C27.2](02-ui-atlases.md)) carries the *imagery* — the pixels of every button, panel, and
  icon.
- **The layout table** carries the *positioning* — where each element sits on screen and which atlas region it
  draws.

A layout entry, conceptually, is: *this element, at this screen position/size, drawn from this atlas
rectangle, showing this label*. The scene ([C27.1](01-shell-scenes.md)) is a collection of these entries; the
shell engine walks them and draws each.

```
LayoutEntry:
  screen_rect  (x, y, w, h)        — where on screen
  atlas_region (u0, v0, u1, v1)    — which atlas pixels (C27.2)
  label        (optional)          — which text (C27.5)
  element_type (button/panel/text/…)
```

## The pattern is everywhere

This atlas + table split is not front-end-specific — it's the UI idiom of the whole engine:

| System | Atlas (imagery) | Table (positioning) |
|---|---|---|
| Front-end / HUD | UI TPK atlases ([C27.2](02-ui-atlases.md)) | layout entries (this page) |
| Fonts | glyph atlas | glyph table ([C28.1](../C28-Fonts-Glyphs/01-atlas-glyph-table.md)) |
| Minimap | tile textures | map data ([C29.1](../C29-Minimap-Map-Data/01-tiles.md)) |

In every case: **a texture holds the pixels; a small table says where things go and which pixels each uses.**
Recognising this one pattern lets you read fonts, HUD, menus, and the minimap as variations of the same idea
([C28.4](../C28-Fonts-Glyphs/04-rendering.md)).

## Why separate imagery from positioning

The separation buys the same independence the whole book keeps returning to:

- **Re-skin without re-laying-out.** Change the atlas art ([C27.2](02-ui-atlases.md)) and the layout still
  positions the (now different-looking) elements correctly.
- **Re-lay-out without re-authoring art.** Move a button by changing its layout entry's screen rectangle,
  reusing the same atlas image.
- **Localize without either.** Text is a label ([C27.5](05-ui-text.md)), so translating changes only the
  string table, not the layout or the art.

Three independent axes — art, position, text — is what makes the UI flexible and localizable, and it's exactly
the atlas + table split that provides it.

## Resolution and anchoring

A layout must work across screen resolutions, so positions are typically expressed in a way that adapts —
relative coordinates or anchors rather than raw pixels — so an element stays in the right place whether the
screen is 640×480 or 1920×1080. This is a standard UI-layout concern; the practical point for editing is that a
layout entry's position may be resolution-relative, so moving an element means adjusting its (possibly
normalized) coordinates, not just pixel offsets.

> 🟡 *Reasoned:* the atlas + layout-table model and layout-entry fields are the standard UI pattern, consistent
> with the verified atlas (TPK) and label systems and the identical font/minimap tables; the precise on-disk
> layout-record byte format is the front-end's detail.

## Editing implications

- **Move/resize elements via the layout table** — change an entry's screen rectangle
  ([C27.6](06-editing-ui.md)).
- **Keep atlas regions correct** — an entry's UV rectangle must point at the intended atlas image
  ([C27.2](02-ui-atlases.md)).
- **Respect anchoring** — edit positions in the layout's coordinate convention (relative/anchored), not raw
  pixels, so the UI still adapts to resolution.
- **The three axes are independent** — position (layout), art (atlas), text (label) edit separately.

---

### Key takeaways

- The UI's core pattern is **atlas + layout table**: pixels in the atlas, positioning in the table.
- A layout entry ties a screen rectangle to an atlas region (and optionally a label) per element.
- The same atlas + table split runs the front-end, HUD, **fonts**, and **minimap** — one idiom, four systems.
- Separating imagery from positioning gives three independent axes: art, position, text.
- Edit positions via the layout table (in its coordinate convention), keep UV regions correct, and localize via
  labels.

**Continue:** [C27.4 — The HUD](04-hud.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md)
