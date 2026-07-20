# C70.2 — Wheels, Brakes & Aero

> **The one-sentence version:** wheels, brakes, and aero are all mesh choices — `RIM` selects the wheel model, the
> front brake/tyre are per-kit geometry, and `SPOILER`/`HOOD`/`ROOFSCOOP` are swappable body pieces — each an
> appearance part the renderer draws into a visual slot.

[← C70.1 — Body kits as geometry](01-body-kits.md) · [Chapter 70 hub](C70-Visual-Customisation.md) ·
[Next: C70.3 — Paint & colour →](03-paint.md)

---

## Wheels

The most-swapped visual part is the **wheel** — `CustomizeRims`
([C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)). A rim is a **mesh** selected into the car's
four wheel positions, the same geometry-swap idea as a body kit ([C70.1](01-body-kits.md)) but shared across cars: a
rim design fits any car, so rims are their own library rather than per-car `GEOMETRY.BIN` content. Choosing a rim
sets which wheel mesh the renderer instances at each corner; the rim can also be recoloured
([C70.3](03-paint.md) — `RIM_PAINT`).

> ✅ *Verified:* `RIM` (×6) and `Rim` (×3) are strings in `speed.exe`; `CustomizeRims` is the wheel category
> ([C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)).

## Brakes and tyres

The wheels-area geometry travels with the body kit ([C70.1](01-body-kits.md)) — the retail car files carry
`KIT00_FRONT_BRAKE_*` and `KIT00_FRONT_TIRE_*` solids ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)). So
the *visible* brake (the caliper/rotor you see behind the rim) and the tyre are meshes bundled with the kit, sized to
fit that body. Note the distinction from the *performance* brake and tyre
([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)): the `PART_BR_*` / `PART_TI_*` parts change braking
and grip *numbers* (the sim, [Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)),
while the `FRONT_BRAKE` / `FRONT_TIRE` meshes are what you *see*. A car has both: a performance brake (a vault value)
and a brake mesh (geometry) — the same real-world object split across the two customizations
([C56.1](../C56-Customization/01-two-customizations.md)).

## Aero: spoilers, hoods, scoops

Aero parts are **swappable body pieces** — `SPOILER` (×9), `HOOD` (×5), `ROOFSCOOP` (×1) in `speed.exe`, with
`CustomizeSpoiler` its own category ([C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)). Like body
kits, each is a mesh the renderer draws in place of (or in addition to) the base piece: choose a spoiler and its mesh
replaces the stock rear deck. These sit on top of the kit — a body kit sets the shell, and the spoiler/hood/scoop are
finer swaps within it. The spoiler is the most-customised aero piece (nine variants named), reflecting how central a
big rear wing is to the tuner look.

> 🟡 *Reasoned:* that wheels are a shared cross-car rim library while brakes/tyres/aero meshes are per-car
> (`KIT00_FRONT_*`, `SPOILER`/`HOOD`/`ROOFSCOOP`) follows from the retail object naming and the `Customize*`
> categories; the exact rim-library location is separate car-parts data. The `RIM`/`SPOILER`/`HOOD`/`ROOFSCOOP`
> strings and the per-kit front geometry are verified.

## All the same idea

Wheels, brakes, and aero unify under one principle: **appearance is mesh selection**. Where performance
customization ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)) picks *numbers*,
visual customization picks *meshes* — the rim mesh, the brake mesh, the spoiler mesh — and the renderer
([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) draws whatever's currently selected. This is the visual
slot model ([C70.1](01-body-kits.md)) applied part by part: the body kit is the big swap, and wheels/brakes/aero are
the finer swaps layered on it. Reading them is reading a *tree of mesh choices* — pick a body, then a wheel, then a
spoiler — each narrowing the car's look, all resolved to solid objects
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) the renderer instances.

## RE implications

- **Wheels** — `RIM` selects a wheel mesh (a shared cross-car library) into the four corners; recolourable via
  `RIM_PAINT` ([C70.3](03-paint.md)).
- **Brakes/tyres** — the *visible* brake/tyre are per-kit meshes (`KIT00_FRONT_*`), distinct from the *performance*
  `PART_BR_*`/`PART_TI_*` (sim numbers).
- **Aero** — `SPOILER`/`HOOD`/`ROOFSCOOP` are swappable body-piece meshes; `CustomizeSpoiler` is its own category.
- **All mesh selection** — visual customization picks meshes the renderer draws; the visual slot model applied part
  by part.

---

### Key takeaways

- **Wheels** are mesh choices (`RIM`, `CustomizeRims`) instanced at the four corners — a **shared cross-car library**
  (a rim fits any car), recolourable separately (`RIM_PAINT`, [C70.3](03-paint.md)).
- **Brakes and tyres** have a **visible** side (per-kit `KIT00_FRONT_BRAKE`/`FRONT_TIRE` meshes) distinct from the
  **performance** side (`PART_BR_*`/`PART_TI_*` sim numbers) — the same object split across the two customizations.
- **Aero** — `SPOILER` (×9), `HOOD` (×5), `ROOFSCOOP` — are **swappable body-piece meshes**; the spoiler is the
  most-customised aero part.
- Everything here is **mesh selection** — appearance is picking meshes the renderer
  ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) draws, the visual slot model
  ([C70.1](01-body-kits.md)) applied part by part.

**Continue:** [C70.3 — Paint & colour](03-paint.md) · [Chapter 70 hub](C70-Visual-Customisation.md)
