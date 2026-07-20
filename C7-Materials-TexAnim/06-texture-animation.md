# C7.6 — Texture Animation

> **The one-sentence version:** MW makes surfaces appear to move by animating the *reference* — scrolling a
> group's UVs, or binding a runtime-updated image from the **DYNAMIC** texture pack (`Global\DynamicTextures.tpk`)
> — never by rewriting stored pixels; this page gives the mechanism and is explicit about what is verified
> versus reasoned.

[← C7.5 — The two hash worlds](05-two-hash-worlds.md) · [Chapter 7 hub](C7-Materials-TexAnim.md) ·
[Next: Chapter 8 — Geometry: Solid Lists & Objects →](../C8-Geometry-Solids/C8-Geometry-Solids.md)

---

## Two ways a surface "animates"

Nothing in a TPK changes over time — the stored bytes are static ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)).
Apparent motion comes from changing *how* a static texture is sampled, or from binding a texture whose pixels
the engine itself refreshes each frame. MW uses both:

1. **UV animation (scrolling / flipbook).** The group samples the same texture, but its texture coordinates
   are offset over time — a continuous scroll (road-edge light streaks, waterfalls, conveyor signage) or a
   stepped flip through cells of an atlas (flames, sparks). The pixels never change; the *window* onto them
   moves.
2. **Dynamic textures (runtime-written images).** Some textures are render targets or procedurally updated
   surfaces — the engine draws into them each frame and any group bound to them shows live content
   (reflections, mirrored views, animated HUD elements, damage overlays).

## The DYNAMIC pack is real and verifiable

The second mechanism has a concrete, inspectable artifact. `GLOBAL/DYNTEX.BIN` is a TPK whose internal name
is **`DYNAMIC`** and whose source path is **`Global\DynamicTextures.tpk`** — verifiable directly from its
info header ([C5.2](../C5-Textures-TPK/02-metadata-tables.md)). It is a large (~10 MB) **compressed-variant**
pack (its entry chunk is `0x33310003`, ~151 per-texture JDLZ blobs), which is exactly what you expect for a
library of textures the engine treats as mutable/animated at runtime. So "dynamic textures" is not a
hypothesis — it is a named, shipped pack, distinct from the static world and car packs.

> ✅ *Verified:* `DYNTEX.BIN` is the TPK `DYNAMIC` / `Global\DynamicTextures.tpk`, compressed variant, the
> engine's channel for runtime-updated textures.

## How UV animation is expressed

UV animation is a property of the **material**, not the pixels: the group's material parameters
([C7.2](02-shading-groups.md)) carry the animation's rate and mode, and the engine advances the UV offset
each frame accordingly. Conceptually:

```
sample_uv(t) = base_uv + velocity * t          # scroll
frame(t)     = floor((t * fps) % num_cells)     # flipbook: pick an atlas cell
```

For a scroll, a per-material velocity moves the coordinates smoothly; for a flipbook, a rate steps through
sub-rectangles of an atlas texture so a sequence of cells plays like frames of film. Because the binding is
by key ([C7.3](03-texture-binding.md)), the *which texture* stays fixed while the *where in it* changes.

> 🟡 *Reasoned (mechanism, not byte-verified here):* the exact material fields that encode scroll velocity or
> flipbook rate are not pinned to specific offsets in this chapter. The *model* — animation lives in material
> parameters and moves UVs over a static texture — is the reliable takeaway; treat any specific offset for
> an animation rate as unconfirmed until decoded against a known animated surface.

## Editing animated surfaces

- **To change what a scrolling surface shows,** edit the *texture* it samples ([C5.5](../C5-Textures-TPK/05-extract-replace.md))
  — the animation (the UV motion) is independent of the pixels and keeps working.
- **To change the motion,** you must alter the material parameters that drive it, not the texture. If you only
  swap pixels, the surface shows new art moving at the old rate.
- **Dynamic textures are engine-written,** so editing their stored bytes in `DYNTEX.BIN` affects only the
  initial/default content; whatever the engine renders into them at runtime overrides it. Do not expect a
  static edit to a render-target texture to persist on screen.
- **Preserve keys either way.** A scrolling or dynamic group still binds by asset key; keep keys stable so the
  binding survives your edit ([C7.3](03-texture-binding.md)).

## Where this connects

Texture animation sits at the junction of three systems: the **material** that stores the animation
([C7.2](02-shading-groups.md)), the **texture** it samples ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[6](../C6-Texture-Codecs/C6-Texture-Codecs.md)),
and the **binding** that connects them ([C7.3](03-texture-binding.md)). Understanding it is mostly
understanding that MW keeps pixels static and animates the reference — a design that keeps the texture
pipeline simple and makes even animated surfaces re-skinnable by the same key-preserving trick as everything
else.

---

### Key takeaways

- MW animates the *reference*, not stored pixels: UV scroll/flipbook, or runtime-written **dynamic** textures.
- The `DYNAMIC` pack (`DYNTEX.BIN` = `Global\DynamicTextures.tpk`, compressed, ~151 textures) is the verified
  channel for runtime-updated textures.
- UV animation is a **material** property (rate/velocity) that moves coordinates over a fixed texture; the
  exact encoding offsets are left unconfirmed here.
- Edit pixels to change what shows; edit material params to change the motion; dynamic-texture edits are only
  defaults.
- Animated surfaces still bind by asset key, so key-preservation re-skins them like any other group.

**Continue:** [Chapter 8 — 3D Geometry: Solid Lists & Objects](../C8-Geometry-Solids/C8-Geometry-Solids.md) ·
[Chapter 7 hub](C7-Materials-TexAnim.md)
