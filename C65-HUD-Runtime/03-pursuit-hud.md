# C65.3 — Placement & Anchoring

> **The one-sentence version:** a HUD element's position is a layout record holding an `anchor` and `size` in
> **normalized `0..1` screen space** — `(0.5,0.5)` is center, `(1,1)` full extent — which the renderer multiplies
> by the actual backbuffer size, so one layout serves every resolution.

[← C65.2 — The HUD object model](02-gauge-cluster.md) · [Chapter 65 hub](C65-HUD-Runtime.md) ·
[Next: C65.4 — Resolution & widescreen →](04-resolution-widescreen.md)

---

## The layout model: pixels vs. positioning

FEng follows one engine-wide idiom ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)): **the atlas
carries the pixels; a layout record carries the positioning.** A HUD element doesn't bake its screen rectangle into
its texture — it *references* an atlas image (by hash, [C65.2](02-gauge-cluster.md)) and *separately* holds a
placement:

```
HUDElement layout record (~32-byte stride):
  nameHash   u32     — asset hash → which atlas image this element draws
  name       ASCII   — optional literal name
  anchor     vec2    — screen POSITION, normalized 0..1
  size       vec2    — screen EXTENT,   normalized 0..1
  rotation   f32     — radians
  flags      u32
```

So *where* a widget is (`anchor`/`size`) is data separate from *what* it looks like (the atlas image). Move a
widget by editing its layout record; re-skin it by editing the atlas — the two are orthogonal
([below](#three-independent-axes)).

> 🟡 *Reasoned:* the ~32-byte layout record (name-hash + normalized anchor/size + rotation + flags) is decoded from
> the FE data by stride heuristic and is consistent with the verified atlas-plus-layout idiom
> ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)); the exact *layout chunk ID* is an open edge
> ([C65.8](08-reading-hud.md)), which is why on-disk repositioning is gated on confirming it. The atlas-by-hash
> reference and normalized-space model are the verified engine idiom.

## Normalized screen space: the key rule

The **critical placement rule**: `anchor` and `size` are **normalized fractions of the backbuffer**, not pixels:

- **`(0.5, 0.5)`** = the **center** of the screen.
- **`(0, 0)`** = top-left; **`(1, 1)`** = bottom-right / full extent.
- **The renderer multiplies by the actual backbuffer size** at draw time: `pixel = normalized × backbufferSize`.

So a widget anchored at `(0.9, 0.9)` sits near the bottom-right corner *at every resolution* — at 640×480 that's
`(576, 432)`; at 1920×1080 it's `(1728, 972)`; the *fraction* is fixed, the pixels scale. This is how **one layout
serves every resolution** ([C65.4](04-resolution-widescreen.md)) — the normalized coordinates are resolution-independent
by construction. The classic mistake is treating these as pixels: a `(0.5, 0.5)` read as pixels would collapse the
whole HUD into the top-left corner. **Normalized space is the anchor of the whole placement system.**

## Three independent axes

Because pixels, positioning, and text are separate ([above](#the-layout-model-pixels-vs-positioning)), a HUD
element edits along **three independent axes**:

| Axis | Edit | Because |
|---|---|---|
| **Position** | the layout `anchor`/`size` (normalized) | placement is a separate record |
| **Art** | the atlas pixels ([C65.5](05-gauges-meters.md)) | reference is a *hash*, not a baked UV |
| **Text** | the label's string table ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) | layouts are language-neutral |

So you can **move/resize** a widget without touching its art (edit the layout), **re-skin** it without moving it
(edit the atlas — positions untouched because the reference is a hash), and **re-word** it without either (edit the
label). These three edits never interfere — the classic separation of *layout*, *art*, and *content*. This is why
HUD modding ([C65.8](08-reading-hud.md)) is precise: to reposition the speedometer you touch only its anchor; to
recolor it you touch only its atlas; to change its units label you touch only the string.

## Anchoring semantics

The **`anchor`** is the widget's *reference point* on screen ([above](#normalized-screen-space-the-key-rule)) — and
anchoring is what makes placement *robust* across resolutions and aspect ratios ([C65.4](04-resolution-widescreen.md)):

- **Corner-anchored elements stay in their corner.** A minimap anchored bottom-left (`~(0.15, 0.85)`) stays
  bottom-left at any resolution — the fraction pins it there.
- **Center-anchored elements stay centered.** The `321_GO` countdown, anchored `~(0.5, 0.5)`, stays centered — it's
  a screen-center overlay regardless of resolution.
- **The `size`** scales the widget proportionally — a gauge sized `(0.2, 0.2)` is a fifth of the screen at any
  resolution, so it keeps its relative prominence.

So anchoring is *proportional placement* — each widget occupies the *same fraction* of the screen everywhere, which
is exactly what a HUD wants (the speedometer should be the same relative size and corner on every display). This
proportional model is why the HUD *looks right* at 4:3 and 16:9 and at every resolution — the layout is expressed in
the one coordinate system (normalized fractions) that's invariant to display size. The only thing normalized space
*doesn't* fully solve is *aspect ratio* (a `(0.2,0.2)` square is a different shape at 4:3 vs 16:9) — which is why
aspect is handled by *separate layout packages* ([C65.4](04-resolution-widescreen.md)), not per-element math.

## RE implications

- **Layout record** — name-hash (art) + normalized `anchor`/`size` + rotation + flags — placement separate from
  pixels.
- **Normalized `0..1` screen space** — `× backbuffer` at draw time — one layout, every resolution (treating as
  pixels is the classic bug).
- **Three independent axes** — position (layout), art (atlas by hash), text (label) — edit without interference.
- **Anchoring is proportional** — each widget holds the same screen fraction everywhere; aspect handled by package
  selection ([C65.4](04-resolution-widescreen.md)).

---

### Key takeaways

- A HUD element's position is a **layout record** — a name-hash (which atlas image) + **`anchor`/`size` in
  normalized `0..1` screen space** + rotation + flags — placement kept **separate from pixels**.
- **Normalized space is the key rule**: `(0.5,0.5)` = center, `(1,1)` = full extent; the renderer does
  `pixel = normalized × backbufferSize`, so **one layout serves every resolution** (reading these as pixels
  collapses the HUD to the corner).
- Elements edit on **three independent axes** — **position** (the layout record), **art** (the atlas, by hash),
  and **text** (the label) — none interfering.
- **Anchoring is proportional** — each widget holds the **same screen fraction** everywhere, so the HUD looks right
  at any resolution; a corner-anchored minimap stays in its corner, a center-anchored countdown stays centered.
- Normalized space handles *resolution* fully; **aspect ratio** is handled by **separate layout packages**
  ([C65.4](04-resolution-widescreen.md)), not per-element math — and the exact **layout chunk ID** is the one open
  edge for trusted on-disk repositioning.

**Continue:** [C65.4 — Resolution & widescreen](04-resolution-widescreen.md) · [Chapter 65 hub](C65-HUD-Runtime.md)
