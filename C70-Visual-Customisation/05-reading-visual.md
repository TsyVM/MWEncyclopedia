# C70.5 — Reading Visual Customisation in RE

> **The one-sentence version:** navigate visual customization by the per-car file set — `GEOMETRY.BIN` (body kits,
> wheels, aero), `TEXTURES.BIN` (base skin), `VINYLS.BIN` (vinyl palette), `PREVINYL.BIN` (baked livery) — plus the
> paint targets, reading a custom car as geometry + colour + texture composed into one appearance.

[← C70.4 — Vinyls & decals](04-vinyls-decals.md) · [Chapter 70 hub](C70-Visual-Customisation.md) ·
[Next: Chapter 71 — Cars, End to End →](../C71-Cars-End-To-End/C71-Cars-End-To-End.md)

---

## The per-car file set

A car's *appearance* is spread across four `BIN` files, each a format the book has already decoded:

| File | Format | Holds |
|---|---|---|
| `GEOMETRY.BIN` | solid objects ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) | the `BASE` + `KIT` body meshes, wheels-area, decal-mount meshes ([C70.1](01-body-kits.md)) |
| `TEXTURES.BIN` | texture pack ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) | the car's base skin (paint-ready surfaces, materials) |
| `VINYLS.BIN` | texture pack ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) | the vinyl/decal artwork **palette** ([C70.4](04-vinyls-decals.md)) |
| `PREVINYL.BIN` | texture pack ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) | the **pre-composited** livery ([C70.4](04-vinyls-decals.md)) |

So reading a car's visuals is reading these four files with the formats from earlier chapters — no new container
format, just the solid-list and `TPK` readers applied to car data. The customization *choices* (which kit, which
paint, which vinyls) select *within* these files; the files are the palette, the choices the selection.

> ✅ *Verified:* the per-car set `GEOMETRY.BIN` (solids), `TEXTURES.BIN` / `VINYLS.BIN` / `PREVINYL.BIN` (`TPK`s) is
> present in retail car folders; each parses with the formats of [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)
> and [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md).

## The RE workflow

Reading visual customization:

1. **Read the geometry** — `GEOMETRY.BIN`; the `BASE`/`KIT` bodies, wheels, aero ([C70.1](01-body-kits.md)–[C70.2](02-wheels-aero.md)).
2. **Read the textures** — `TEXTURES.BIN` (base skin) + `VINYLS.BIN` (vinyl palette) ([C70.4](04-vinyls-decals.md)).
3. **Read the paint targets** — `BASE_PAINT`/`RIM_PAINT`/`VINYL_PAINT`/`HUD_PAINT` ([C70.3](03-paint.md)).
4. **See the composite** — `PREVINYL.BIN`, the baked livery ([C70.4](04-vinyls-decals.md)).

The output is the full visual picture: a car is a chosen body (geometry) + a paint (colour) + a baked livery
(texture), composed by the renderer ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)).

## The two consumers, revisited

This chapter is the *renderer's* half of the car ([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)):

- **The sim** reads the **vault** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — performance
  ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)).
- **The renderer** reads the **geometry + textures + paint** (this chapter) — appearance.

Both are configured through the same object and shop ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)),
but they reach *different data*: performance customization edits vault numbers, visual customization selects meshes,
textures, and colours. This is the deepest structural fact of MW's cars: **one object, two data domains, two
consumers** — and it's why you can make a slow car look fast or a fast car look stock. Reading a car completely means
reading both domains: the vault (performance) and the file set (visual).

## Where the cluster goes next

- **[Chapter 71 — Cars, End to End](../C71-Cars-End-To-End/C71-Cars-End-To-End.md)** — the *walkthrough* that ties
  the object ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)), performance
  ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)), and visuals (this chapter)
  into one complete car-modding pass — from picking a `CarType` to a fully-built, painted, liveried car.

## RE implications

- **The per-car file set** — `GEOMETRY.BIN` (solids) + `TEXTURES.BIN`/`VINYLS.BIN`/`PREVINYL.BIN` (`TPK`s), read with
  [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md) / [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md).
- **The RE workflow** — geometry → textures → paint targets → composite.
- **Two consumers** — sim reads the vault (performance), renderer reads the file set (visual); one object, two data
  domains.
- **Next** — [Chapter 71](../C71-Cars-End-To-End/C71-Cars-End-To-End.md) ties object + performance + visual into one
  walkthrough.

---

### Key takeaways

- A car's appearance is **four `BIN` files** — `GEOMETRY.BIN` (solids, [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)),
  `TEXTURES.BIN`/`VINYLS.BIN`/`PREVINYL.BIN` (`TPK`s, [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) — **no new
  format**, just the solid-list and `TPK` readers on car data.
- The RE workflow: **geometry → textures → paint targets → composite** — a car is a chosen **body** + **paint** +
  baked **livery**, composed by the renderer ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)).
- This is the **renderer's half** of the car — visual customization selects meshes/textures/colours, while
  performance ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)) edits vault
  numbers: **one object, two data domains, two consumers**.
- Reading a car **completely** means both domains — the **vault** (performance,
  [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and the **file set** (visual, this chapter).
- The cars cluster closes with [Chapter 71](../C71-Cars-End-To-End/C71-Cars-End-To-End.md) — the end-to-end
  walkthrough.

**Next:** [Chapter 71 — Cars, End to End](../C71-Cars-End-To-End/C71-Cars-End-To-End.md).

**Sources:** `speed.exe` (verified strings: `BODYKIT`, `KIT`, `SPOILER`, `HOOD`, `ROOFSCOOP`, `RIM`, `VINYL`,
`DECAL`, `PAINT`, `GLOSS`; paint targets `BASE_PAINT`/`RIM_PAINT`/`VINYL_PAINT`/`VISUAL_PAINT`/`HUD_PAINT`; the
`Customize*` visual categories). Retail car data: `CARS/<car>/GEOMETRY.BIN` (`BASE`+`KIT00_*` solids),
`TEXTURES.BIN`/`VINYLS.BIN` (276 textures)/`PREVINYL.BIN` (3 textures) as `VINYL` `TPK`s. Formats:
[Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md) (solids), [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)
(`TPK`), [Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md) (materials/paint).
