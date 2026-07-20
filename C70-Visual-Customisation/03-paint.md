# C70.3 — Paint & Colour

> **The one-sentence version:** paint is a *multi-target* recolour — the game paints the body (`BASE_PAINT`), the
> rims (`RIM_PAINT`), the vinyls (`VINYL_PAINT`), and the HUD (`HUD_PAINT`) as separate targets, each a colour and a
> finish (`GLOSS`) the renderer applies to surfaces, not new geometry.

[← C70.2 — Wheels, brakes & aero](02-wheels-aero.md) · [Chapter 70 hub](C70-Visual-Customisation.md) ·
[Next: C70.4 — Vinyls & decals →](04-vinyls-decals.md)

---

## Paint is a material, not geometry

Unlike body kits ([C70.1](01-body-kits.md)) and wheels ([C70.2](02-wheels-aero.md)), paint adds *no geometry* — it
**recolours existing surfaces**. Colour is a **material parameter** ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md))
the renderer ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) applies when drawing the car's meshes: the
same panels, a different tint. `CustomizePaint` is the category, and `CustomizePaintDatum` is *a colour entry* — the
specific colour value written to the car ([C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)). So
painting is the cheapest customization to store (a colour, not a mesh or texture) and the fastest to apply (a shader
parameter).

> ✅ *Verified:* `PAINT` (×8), `Paint` (×7), and `GLOSS` (×3) are strings in `speed.exe`, with the paint targets
> `BASE_PAINT`, `RIM_PAINT`, `VINYL_PAINT`, `VISUAL_PAINT`, `HUD_PAINT`; `CustomizePaint` / `CustomizePaintDatum` are
> the paint category and colour entry.

## The paint targets

Paint is not one colour but **several, one per target surface**:

| Target | Paints |
|---|---|
| `BASE_PAINT` | the car body (the main colour) |
| `RIM_PAINT` | the wheels ([C70.2](02-wheels-aero.md)) |
| `VINYL_PAINT` | the vinyls/decals ([C70.4](04-vinyls-decals.md)) |
| `HUD_PAINT` | the player's HUD tint ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) |
| `VISUAL_PAINT` | the overall visual-customization colour context |

So a car carries *multiple* paint values — the body one colour, the rims another, the vinyls a third — each applied
to its own surfaces independently. This is why MW lets you colour a wheel differently from the body, or tint a decal
without repainting the car: they're *separate paint targets*, not one global colour. The `HUD_PAINT` target extends
the idea past the car to the *UI* ([C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)) — your chosen
colour tints your on-screen HUD, a cosmetic identity that follows you into the race.

## Colour and finish

Each paint target carries not just a hue but a **finish** — `GLOSS` names the finish dimension. A colour is
therefore a small tuple: which colour, how glossy (and, on the visual side, metallic/pearl variants the material
supports, [Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)). The renderer reads this when shading the
target's surfaces, so a "candy red gloss" body and a "matte black" wheel are the same mechanism with different
colour+finish tuples on different targets. This keeps paint *data-light* — a few values per target — while giving the
range of looks players expect, because the *material shader* does the visual work from those few parameters.

> 🟡 *Reasoned:* the colour+finish tuple per target (hue plus `GLOSS`/metallic/pearl) is the natural reading of the
> paint-target strings and the material system ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)); the
> exact stored paint record is vault/save data. The paint targets and `GLOSS` string are verified.

## Where paint sits in the visual stack

Paint is the *last* layer of the visual build — it recolours whatever geometry and vinyls are already chosen:

```
body kit (C70.1)  ->  wheels/aero (C70.2)  ->  vinyls (C70.4)  ->  PAINT recolours all of it
```

Because paint is a parameter over the *current* surfaces, it composes cleanly with everything upstream: swap the body
kit and the `BASE_PAINT` still applies to the new panels; add a vinyl and `VINYL_PAINT` tints it. This ordering — mesh
first, texture next, colour last — mirrors a real paint shop (build the car, apply graphics, then paint) and is why
the customizations feel independent: paint doesn't care *what* geometry or vinyl it's colouring, only *which target*
it's assigned to. Reading paint is reading the recolour layer that sits atop the whole visual build
([C70.5](05-reading-visual.md)).

## RE implications

- **Paint is a material parameter** ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)), not geometry — a
  recolour the renderer applies; cheap to store (a colour), fast to apply (a shader parameter).
- **Multi-target** — `BASE_PAINT` (body), `RIM_PAINT` (wheels), `VINYL_PAINT` (decals), `HUD_PAINT` (UI),
  `VISUAL_PAINT` (context) — separately paintable surfaces.
- **Colour + finish** — a hue plus `GLOSS`/metallic/pearl; the material shader does the work from a few parameters.
- **The last layer** — paint recolours whatever geometry/vinyls are chosen; composes cleanly atop the visual build.

---

### Key takeaways

- Paint adds **no geometry** — it **recolours existing surfaces** via a **material parameter**
  ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) the renderer applies; `CustomizePaintDatum` is a
  colour entry, the cheapest customization to store.
- Paint is **multi-target** — **`BASE_PAINT`** (body), **`RIM_PAINT`** (wheels), **`VINYL_PAINT`** (decals),
  **`HUD_PAINT`** (UI), **`VISUAL_PAINT`** (context) — so a car carries **several independent paint values**, not one
  global colour.
- Each target carries a **colour + finish** (`GLOSS`, plus metallic/pearl) — a small tuple the **material shader**
  turns into the look, keeping paint data-light.
- Paint is the **last layer** of the visual stack — mesh → texture → **colour** — so it composes cleanly atop
  whatever body kit, wheels, and vinyls are chosen.
- Verified: `PAINT`/`GLOSS` and the five paint targets in `speed.exe`; `CustomizePaint`/`CustomizePaintDatum`.

**Continue:** [C70.4 — Vinyls & decals](04-vinyls-decals.md) · [Chapter 70 hub](C70-Visual-Customisation.md)
