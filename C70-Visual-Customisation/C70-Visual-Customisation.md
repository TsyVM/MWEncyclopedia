# Chapter 70 — Visual Customisation

> **Goal of this chapter:** decode the *cosmetic* half of customization — the body kits stored as alternate
> geometry (`GEOMETRY.BIN` `BASE` + `KIT` variants), the wheels/brakes/aero (`RIM`, `SPOILER`, `HOOD`,
> `ROOFSCOOP`), the multi-target paint (`BASE_PAINT`/`RIM_PAINT`/`VINYL_PAINT`/`HUD_PAINT`), and the vinyls/decals
> stored as the per-car `VINYLS.BIN` / `PREVINYL.BIN` texture packs.

Chapter 68 gave the *object and shop* ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md));
Chapter 69 the *performance mechanics* ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)).
This chapter decodes the *visual* side — the second of the two customizations
([C56.1](../C56-Customization/01-two-customizations.md)) — where the same slot/shop machinery drives *appearance*
instead of behaviour. Its distinguishing depth is the **per-car file set**: the actual `GEOMETRY.BIN`,
`TEXTURES.BIN`, `VINYLS.BIN`, and `PREVINYL.BIN` a car ships, and how body kits, wheels, paint, and decals live in
them.

> **Verified against the executable and retail car files.** The visual categories are the `Customize*` screens
> ([C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)): `CustomizePaint`, `CustomizeRims`,
> `CustomizeSpoiler`, `CustomizeDecals`, `CustomizeNumbers`, `CustomizeHUDColor`. `speed.exe` also names
> **`BODYKIT`**, **`KIT`** (×8), **`SPOILER`** (×9), **`HOOD`** (×5), **`ROOFSCOOP`**, **`RIM`** (×6), **`VINYL`**
> (×13), **`DECAL`** (×94), **`PAINT`** (×8) with the paint targets **`BASE_PAINT`**, **`RIM_PAINT`**,
> **`VINYL_PAINT`**, **`VISUAL_PAINT`**, **`HUD_PAINT`**, and **`GLOSS`**. Retail car data (e.g.
> `CARS/BMWM3GTR/GEOMETRY.BIN`) stores body geometry as a `BASE` set plus `KIT00_*` variants
> (`KIT00_BODY`/`DAMAGE`/`DECAL`/`DRIVER`/`FRONT`); `VINYLS.BIN` is a `VINYL` texture pack (276 textures),
> `PREVINYL.BIN` its pre-composited form (3 textures).

---

## Deep-dive pages

- [C70.1 — Body kits as geometry](01-body-kits.md): `GEOMETRY.BIN` `BASE` + `KIT` variants — body kits are alternate
  meshes.
- [C70.2 — Wheels, brakes & aero](02-wheels-aero.md): `RIM`, front brake/tyre geometry, and the `SPOILER`/`HOOD`/
  `ROOFSCOOP` swaps.
- [C70.3 — Paint & colour](03-paint.md): the multi-target paint system (`BASE_PAINT`/`RIM_PAINT`/`VINYL_PAINT`/
  `HUD_PAINT`) and `GLOSS`.
- [C70.4 — Vinyls & decals](04-vinyls-decals.md): the `VINYLS.BIN` texture library, `PREVINYL.BIN` compositing, and
  the `DECAL` system.
- [C70.5 — Reading visual customisation in RE](05-reading-visual.md): the per-car file set and how it composes a
  car.

---

## 70.1 Body kits as geometry

A body kit is **alternate geometry** ([C70.1](01-body-kits.md)), not a parameter: the car's `GEOMETRY.BIN`
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) stores a `BASE` body plus one or more `KIT` variants
(`KIT00_BODY_A..E`, with their own damage stages, decal meshes, and front parts). Choosing a body kit *swaps which
solid objects the car renders* — the visual analogue of the performance slot
([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)), but the "part" is a mesh set.

## 70.2 Wheels, brakes & aero

Wheels, brakes, and aero are the same idea ([C70.2](02-wheels-aero.md)): `RIM` selects the wheel mesh, the front
brake/tyre are geometry (`KIT00_FRONT_BRAKE`/`FRONT_TIRE`), and `SPOILER`/`HOOD`/`ROOFSCOOP` are swappable body
pieces. Each is a mesh choice the renderer draws — appearance parts that fill visual slots.

## 70.3 Paint & colour

Paint is **multi-target** ([C70.3](03-paint.md)): the game paints the body (`BASE_PAINT`), the rims (`RIM_PAINT`),
the vinyls (`VINYL_PAINT`), and even the HUD (`HUD_PAINT`) as separate targets, each with a colour and a finish
(`GLOSS`). Colour is a *material parameter* the renderer ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md))
applies to the car's surfaces — not new geometry, but a recolour of what's there.

## 70.4 Vinyls & decals

Vinyls and decals are **textures** ([C70.4](04-vinyls-decals.md)): the per-car `VINYLS.BIN` is a texture pack
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) of the vinyl artwork (276 textures for the M3 GTR), and
`PREVINYL.BIN` holds the *pre-composited* result (3 textures) — the game bakes your chosen, positioned vinyls into a
compact texture the car wears. `DECAL` (×94) is the decal machinery; a livery is layered vinyl textures composited
onto the body.

---

### Key takeaways

- Visual customization is the **second customization** ([C56.1](../C56-Customization/01-two-customizations.md)) —
  same slot/shop machinery ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)),
  but the "parts" are **meshes, textures, and colours** the *renderer* reads.
- **Body kits are alternate geometry** — `GEOMETRY.BIN` stores a `BASE` body plus `KIT` variants
  ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)); choosing a kit swaps the rendered meshes.
- **Wheels/brakes/aero** are mesh choices (`RIM`, front brake/tyre, `SPOILER`/`HOOD`/`ROOFSCOOP`) — appearance parts
  in visual slots.
- **Paint is multi-target** (`BASE_PAINT`/`RIM_PAINT`/`VINYL_PAINT`/`HUD_PAINT`, plus `GLOSS`) — a recolour material
  the renderer applies, not new geometry.
- **Vinyls/decals are textures** — `VINYLS.BIN` is the artwork library, `PREVINYL.BIN` the **pre-composited** bake
  ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) — a livery is layered vinyl textures on the body.

**Next:** [C70.1 — Body kits as geometry](01-body-kits.md).
