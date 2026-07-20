# C27.2 — UI Atlases

> **The one-sentence version:** the UI's imagery — buttons, panels, icons, backgrounds — is packed into TPK
> texture atlases, so many small UI images share a few textures, and re-skinning the interface is editing
> these atlases.

[← C27.1 — The shell: scenes & atlases](01-shell-scenes.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md) ·
[Next: C27.3 — Layout: positioning the UI →](03-layout.md)

---

## UI art is TPK

The pictures the front-end draws are ordinary **TPK** texture atlases
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) — the same texture container as cars and the world. A UI
atlas is one (or a few) TPK textures onto which many small UI images are packed: a sheet of buttons, a sheet of
icons, a background image. The scene's elements ([C27.1](01-shell-scenes.md)) each reference a **region** of an
atlas ([C27.3](03-layout.md)) rather than a whole texture.

Because they're TPKs, everything you learned about textures applies:

- **Decode** with the codec chapter ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) — UI art is DXT or
  ARGB like any texture.
- **Extract / replace** with the TPK editing path ([C5.5](../C5-Textures-TPK/05-extract-replace.md)) — re-skin
  the UI by swapping atlas pixels.
- **Reference by key** — a UI element names a texture the same keyed way materials do
  ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)).

## Why atlases

Packing many UI images into a few atlas textures is standard UI efficiency:

- **Fewer textures, fewer binds.** The GPU binds one atlas and draws many elements from it, instead of binding
  a texture per icon — far cheaper.
- **Shared memory.** One atlas texture holds dozens of small images that would waste space as separate
  textures (each rounded up to a power of two).
- **Batch drawing.** Elements from the same atlas draw in one batch.

This is the same reason the world uses texture packs and the fonts use a glyph atlas
([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) — atlasing is the universal answer to "many small
images."

## Regions, not whole textures

The key idea distinguishing a UI atlas from a plain texture is that elements use **sub-regions**. An atlas is a
grid (or packing) of images, and each UI element's imagery is a **UV rectangle** into it — the same mechanism a
glyph uses ([C28.1](../C28-Fonts-Glyphs/01-atlas-glyph-table.md)) and an animated texture uses
([C7.6](../C7-Materials-TexAnim/06-texture-animation.md)):

```
atlas texture (one TPK image)
┌───────┬───────┬─────────┐
│ btn A │ btn B │  icon   │   each element samples its
├───────┴───────┼─────────┤   own (u0,v0)-(u1,v1) rectangle
│  background   │  panel  │
└───────────────┴─────────┘
```

The layout table ([C27.3](03-layout.md)) stores which rectangle each element uses. So the atlas is the
*imagery*, and the rectangle is the *selection* — imagery and positioning separated, as everywhere in the UI.

## The HUD atlases

The in-game HUD ([C27.4](04-hud.md)) has its own atlases — the `HUDTEX*` and `HUDS_Custom_*` files in
`GLOBAL/` are HUD texture packs (verified as `.BIN`/TPK art in the sound/global data). The speedometer, minimap
frame, heat meter, and pursuit icons are regions of these HUD atlases, drawn by the HUD layout. Same
atlas-and-region model as the menus, applied to the in-game overlay.

## Editing implications

- **Re-skin by editing the atlas TPK** ([C5.5](../C5-Textures-TPK/05-extract-replace.md)) — change how buttons,
  panels, and icons look without touching layouts.
- **Keep regions aligned.** If you move images within an atlas, the layout's UV rectangles must follow, or
  elements sample the wrong art ([C27.3](03-layout.md)).
- **Prefer same-size atlas edits** — same-dimension re-skins are in-place TPK swaps
  ([C5.5](../C5-Textures-TPK/05-extract-replace.md)); resizing the atlas is a repack.
- **Match the format.** Keep the atlas's DXT/ARGB format on replacement
  ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).

---

### Key takeaways

- UI imagery is **TPK atlases** — the same texture container as everything else; decode/edit with Chapters 5–6.
- Many small UI images share a few atlas textures — the standard efficiency of fewer binds and shared memory.
- Elements use **sub-region UV rectangles** into the atlas; imagery (atlas) is separate from selection
  (rectangle).
- The HUD has its own atlases (`HUDTEX*`/`HUDS_Custom_*`) drawn the same way.
- Re-skin by editing the atlas TPK; keep UV regions aligned; prefer same-size edits and matching formats.

**Continue:** [C27.3 — Layout: positioning the UI](03-layout.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md)
