# Chapter 27 — Front-End: Shell Scenes & UI Atlases

> **Goal of this chapter:** decode the menus — the front-end "shell" of scenes, the UI texture atlases they
> draw from, and the layout data that positions everything — so you can read, re-skin, and re-lay-out the
> game's interface.

Everything outside driving — the title, the menus, the garage, the HUD — is the **front-end**, or "shell." It
is a data-driven UI: **scenes** (screens) reference **atlases** (UI textures) and **layout** data (where each
element sits), and the shell engine draws them. This chapter maps that structure. Fonts get their own chapter
([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)); the text the UI shows is localized in
[Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md).

> **Grounded in verified formats.** The front-end ships in `FRONTEND/` (`FRONTA.BUN`, the JDLZ-compressed
> `FrontB.lzc`, and `PLATFORMS/`). The UI is built on the same primitives verified elsewhere in this book: UI
> art is **TPK** atlases ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), the front-end bundle is JDLZ
> ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)), and text is the **label system**
> ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)).

---

## Deep-dive pages

- [C27.1 — The shell: scenes & atlases](01-shell-scenes.md): the front-end as scenes drawing from UI texture
  atlases.
- [C27.2 — UI atlases](02-ui-atlases.md): the TPK art the interface is built from.
- [C27.3 — Layout: positioning the UI](03-layout.md): the atlas + layout-table pattern that places elements.
- [C27.4 — The HUD](04-hud.md): the in-game overlay as a layout over atlases.
- [C27.5 — Text in the UI](05-ui-text.md): how the shell shows localized strings by label.
- [C27.6 — Re-skinning & re-laying-out](06-editing-ui.md): editing atlases and layouts safely.

---

## 27.1 The shell is scenes over atlases

The front-end is a set of **scenes** — the title screen, the main menu, the garage, the map, the results
screens — each a *layout* of UI elements drawn from *atlases* of UI art. A scene says "put this button here,
this panel there, this text in that box," referencing images in the atlases and strings by label
([C27.1](01-shell-scenes.md)). The shell engine walks the active scene and draws it. It is a data-driven UI:
the *screens are data*, and the engine renders them.

## 27.2 UI art is TPK atlases

The images the UI draws — buttons, panels, icons, backgrounds — are packed into **TPK** texture atlases
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), the same texture container as the world and cars. A UI
element references a region of an atlas ([C27.3](03-layout.md)), so many small UI images share a few atlas
textures — the standard atlas efficiency ([C27.2](02-ui-atlases.md)). Re-skinning the interface is, at bottom,
editing these TPK atlases ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).

## 27.3 Layout is atlas + table

The recurring UI pattern is **atlas + layout table**: the atlas carries the *pixels*, and a small *table*
positions elements on the screen and maps each to its atlas region. This is the exact pattern the fonts use
(atlas + glyph table, [Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) and the minimap uses (tiles + map
data, [Chapter 29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)): **imagery in a texture, positioning in a
table** ([C27.3](03-layout.md)). Separating the two lets the layout change without re-authoring art, and the
art change without re-laying-out.

## 27.4 The HUD is a layout too

The in-game HUD — speedometer, minimap, heat meter, pursuit info — is the same idea applied over gameplay: a
**layout** of elements drawn from HUD atlases, updated each frame with live values
([C27.4](04-hud.md)). The `HUDTEX*`/`HUDS_Custom_*` files in `GLOBAL/` are the HUD's atlases; the layout places
them and binds them to the values they display (speed, heat, position).

## 27.5 Text is by label, not literal

The shell never stores literal words in its scenes — it references **labels** ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)):
a scene says "show label `MENU_START`," and the label system resolves it to the active language's string
([C27.5](05-ui-text.md)). This is what makes the UI localizable without re-authoring every screen per language —
the layout is language-neutral, and only the string table changes.

---

### Key takeaways

- The front-end is **scenes** (screens) that lay out UI elements drawn from **atlases** and text by **label**.
- UI art is **TPK atlases** (Chapter 5) — re-skinning the UI is editing these textures.
- The recurring pattern is **atlas + layout table**: pixels in a texture, positioning in a table — shared with
  fonts and the minimap.
- The **HUD** is the same layout-over-atlas idea applied to live gameplay values.
- Text is referenced by **label** (Chapter 30), so scenes are language-neutral and localizable.

**Next:** [Chapter 28 — Fonts & Glyph Tables](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md): the letterforms the UI
renders text with.
