# C27.4 — The HUD

> **The one-sentence version:** the in-game HUD — speedometer, minimap, heat meter, pursuit info — is the same
> atlas + layout pattern as the menus, drawn over gameplay and updated each frame with live values.

[← C27.3 — Layout: positioning the UI](03-layout.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md) ·
[Next: C27.5 — Text in the UI →](05-ui-text.md)

---

## The HUD is UI over gameplay

The **HUD** (heads-up display) is the overlay you see while driving: the speedometer, the minimap
([Chapter 29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)), the heat meter
([C14.2](../C14-Vault-Pursuit-Surfaces/02-heat-bounty.md)), pursuit information, position/lap counters, and
event prompts. Mechanically it is the **same UI** as the menus ([C27.1](01-shell-scenes.md)) — a **layout** of
elements drawn from **atlases** ([C27.2](02-ui-atlases.md)) — but rendered *over* the game and driven by *live*
values.

## HUD atlases

The HUD's imagery is its own set of atlases — the `HUDTEX*` texture packs and the `HUDS_Custom_*` files in
`GLOBAL/` (verified as HUD `.BIN` art). The speedometer face, the minimap frame, the heat/pursuit icons, and
the prompt graphics are regions of these HUD atlases ([C27.2](02-ui-atlases.md)), positioned by the HUD layout
([C27.3](03-layout.md)). Different HUD variants (`HUDS_Custom_00`…`_10`) suggest per-mode/per-configuration HUD
sets.

## Live values, not static text

The one thing that distinguishes the HUD from a menu is that it's **updated every frame** with gameplay values:

```
each frame:
  speed        ← vehicle simulation → speedometer needle/number
  heat         ← pursuit system → heat meter fill (C14.2)
  minimap      ← player position + road network → map overlay (C29)
  pursuit info ← cop system → cop count / bust bar (C49 in the plan)
```

So a HUD element binds an atlas region *and* a **live value source**: the speedometer's needle rotates with
speed, the heat meter fills with heat, the minimap centers on the player. The layout places the element; the
binding feeds it data. This is the menu pattern (atlas + layout) plus a **data binding** to gameplay.

> 🟡 *Reasoned:* the HUD's atlas + layout + live-binding model is the standard in-game-HUD shape, consistent
> with the verified HUD atlases (`HUDTEX*`/`HUDS_Custom_*`) and the gameplay systems it displays (speed, heat,
> minimap); the exact HUD layout-record format is the front-end's detail.

## The minimap is a HUD element

The most complex HUD element is the **minimap** — itself an atlas + table system
([Chapter 29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)): tiles of the map drawn under a frame, with the
player, objectives, and cops overlaid as icons. It's a UI-within-the-UI, positioned by the HUD layout but
internally driven by the world position and the road network
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)). So the HUD nests systems: a layout of
elements, one of which (the minimap) is its own layered display.

## HUD variants and modes

The multiple `HUDS_Custom_*` sets point to the HUD adapting to context — a race HUD (position, lap, checkpoint),
a pursuit HUD (heat, cop count, bust bar), a free-roam HUD. Each mode selects a HUD layout/atlas set
appropriate to what the player needs to see. So "the HUD" is really a family of layouts over shared and
mode-specific atlases, switched by game mode ([the session/mode system, plan Chapter 57](../README.md)).

## Editing implications

- **Re-skin the HUD via its atlases** (`HUDTEX*`/`HUDS_Custom_*`) — change the speedometer face, meter graphics,
  icons ([C27.2](02-ui-atlases.md)).
- **Re-position via the HUD layout** ([C27.3](03-layout.md)) — move the speedometer, resize the minimap.
- **Bindings are data too** — which value feeds which element is part of the HUD data, though changing bindings
  is deeper than re-skinning.
- **Mind the mode variants** — edit the right `HUDS_Custom_*` set for the mode you're targeting.

---

### Key takeaways

- The HUD is the **same UI** as the menus — a layout of elements drawn from atlases — rendered over gameplay.
- HUD imagery is the `HUDTEX*`/`HUDS_Custom_*` atlases; elements are regions positioned by the HUD layout.
- Unlike menus, HUD elements bind to **live gameplay values** (speed, heat, position) updated each frame.
- The **minimap** is a HUD element that is itself an atlas + table system (Chapter 29).
- Multiple `HUDS_Custom_*` sets are **mode variants**; re-skin via atlases, re-position via layout, target the
  right mode's set.

**Continue:** [C27.5 — Text in the UI](05-ui-text.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md)
