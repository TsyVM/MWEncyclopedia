# C71.3 — The Visual Build

> **The one-sentence version:** building appearance layers the visual trinity — choose a body kit (geometry), fit
> wheels and aero, lay a livery of vinyls (baked to `PREVINYL.BIN`), and paint every target — mesh then texture then
> colour, composed by the renderer into the car you see.

[← C71.2 — The performance build](02-performance-build.md) · [Chapter 71 hub](C71-Cars-End-To-End.md) ·
[Next: C71.4 — Modding a car's files →](04-modding-files.md)

---

## The visual build order

Appearance is built in a natural order — mesh first, texture next, colour last
([C70.3](../C70-Visual-Customisation/03-paint.md)):

```
1. body kit      (geometry — swap the shell, C70.1)
2. wheels + aero (geometry — rims, spoiler/hood/scoop, C70.2)
3. vinyls/decals (texture — lay the livery, baked to PREVINYL.BIN, C70.4)
4. paint         (colour — recolour every target, C70.3)
```

The order matters because each layer sits atop the previous ([C70.3](../C70-Visual-Customisation/03-paint.md)): you
can't paint a panel you haven't chosen, or place a decal on a body kit you haven't fitted. Build the shape, apply the
graphics, then colour it — the same order a real shop works, and why the layers compose cleanly.

## Step 1–2: the geometry

First the *shape*. Choosing a **body kit** ([C70.1](../C70-Visual-Customisation/01-body-kits.md)) swaps the car's
active solids — the `BASE` body for a `KIT` variant in `GEOMETRY.BIN`
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) — a whole self-consistent shell (panels, damage geometry,
decal mounts). Then the *details*: **wheels** ([C70.2](../C70-Visual-Customisation/02-wheels-aero.md)) select a rim
mesh at each corner, and **aero** (`SPOILER`/`HOOD`/`ROOFSCOOP`) swaps finer body pieces. After this step the car has
its *form* — the meshes the renderer will draw ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)).

## Step 3: the livery

Next the *graphics*. A livery is layers of **vinyls** ([C70.4](../C70-Visual-Customisation/04-vinyls-decals.md)) —
shapes, logos, tribals from the car's `VINYLS.BIN` palette ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) —
each positioned, scaled, rotated, and coloured on the body, projected onto the `KIT00_DECAL_*` mount meshes
([C70.1](../C70-Visual-Customisation/01-body-kits.md)). When you're done, the game **bakes** the whole stack into the
pre-composited `PREVINYL.BIN` ([C70.4](../C70-Visual-Customisation/04-vinyls-decals.md)) — a single flattened texture
the car wears — so however elaborate the livery, it costs one texture to render. Racing numbers
([C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)) are the same mechanism with glyphs.

## Step 4: the paint

Finally the *colour*. Paint ([C70.3](../C70-Visual-Customisation/03-paint.md)) recolours every target independently:
the body (`BASE_PAINT`), the rims (`RIM_PAINT`), the vinyls (`VINYL_PAINT`), even the HUD (`HUD_PAINT`), each with a
colour and finish (`GLOSS`). Because paint is a material parameter ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md))
over the *current* surfaces, it applies to whatever geometry and vinyls you chose in steps 1–3 — swap the kit and the
paint follows to the new panels. This is the last layer: the car now has form (geometry), graphics (livery), and
colour (paint) — a complete appearance.

## The finished look

Stepping back, the visual build assembles the **trinity** ([C70.4](../C70-Visual-Customisation/04-vinyls-decals.md))
into one car:

- **Geometry** — the chosen body kit, wheels, and aero (`GEOMETRY.BIN` solids).
- **Texture** — the baked livery (`PREVINYL.BIN`) plus the base skin (`TEXTURES.BIN`).
- **Colour** — the per-target paint.

The renderer ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) composes these each frame: draw the active
solids, textured with the base skin and the precomp livery, tinted by the paint targets. The result is *your* car —
and because it's all selections into the shared model ([C71.1](01-anatomy.md)), it's a small per-save delta over the
shared `CarType` data. The visual build is the appearance half of the car; the performance build
([C71.2](02-performance-build.md)) is the behaviour half — together, a complete `PlayerCar`.

## RE implications

- **Build order** — geometry (kit → wheels/aero) → texture (livery) → colour (paint); each layer atop the previous.
- **Geometry** — body kit swaps active solids; wheels/aero are finer mesh swaps.
- **Livery** — layered vinyls baked to `PREVINYL.BIN` (one texture at race time).
- **Paint** — per-target recolour, last layer, follows the current geometry/vinyls.

---

### Key takeaways

- The visual build follows a **natural order** — **geometry** (body kit → wheels/aero) → **texture** (livery) →
  **colour** (paint) — each layer sitting atop the previous, the same order a real shop works.
- **Geometry** swaps active solids — a **body kit** ([C70.1](../C70-Visual-Customisation/01-body-kits.md)) is a whole
  shell, wheels/aero are finer swaps ([C70.2](../C70-Visual-Customisation/02-wheels-aero.md)).
- A **livery** is layered vinyls ([C70.4](../C70-Visual-Customisation/04-vinyls-decals.md)) **baked to
  `PREVINYL.BIN`** — one texture at race time, however elaborate.
- **Paint** is the last layer — per-target recolour ([C70.3](../C70-Visual-Customisation/03-paint.md)) over the
  current surfaces, so it follows whatever geometry/vinyls you chose.
- The finished look is the **trinity** — geometry + texture + colour — composed by the renderer
  ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)); with the performance build
  ([C71.2](02-performance-build.md)), a complete `PlayerCar`.

**Continue:** [C71.4 — Modding a car's files](04-modding-files.md) · [Chapter 71 hub](C71-Cars-End-To-End.md)
