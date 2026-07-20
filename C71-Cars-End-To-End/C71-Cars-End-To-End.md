# Chapter 71 — Cars, End to End

> **Goal of this chapter:** tie every car system in the book into one complete picture — the anatomy of a car
> (object + vault + file set), the performance build (buy → tier → bar → sim), the visual build (kit → wheels → paint
> → livery), and the file-level modding pass — a walkthrough from a bare `CarType` to a fully-built, painted, driven,
> and moddable car.

This is the **capstone** of the cars cluster ([Chapters 68–70](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)).
Where those decoded the *object* ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)),
the *performance mechanics* ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)), and
the *visual layer* ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)), this chapter **assembles
them** — and connects them down to the formats (geometry [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md),
textures [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md), vault [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)),
the sim ([Chapters 39–42](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), and the save
([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)). It's the "everything about cars, in one place"
chapter.

> **Built on verified chapters.** Nothing here is new bytes — it's the synthesis of the verified facts from the
> cluster: the object (`PlayerCar`/`CarType`/`CarSlot`/`CarPart`), the performance catalog (`PART_*` families, the
> `PERF_*` ratings, the `TOPSPEED`/`ACCELERATION`/`HANDLING` bars), and the visual file set (`GEOMETRY.BIN` `BASE`+
> `KIT`, `TEXTURES.BIN`/`VINYLS.BIN`/`PREVINYL.BIN`, the paint targets). Every claim traces to its source chapter.

---

## Deep-dive pages

- [C71.1 — Anatomy of a car](01-anatomy.md): the complete picture — object, two data domains, and the file set.
- [C71.2 — The performance build](02-performance-build.md): buy → tier → bar → sim, as a walkthrough.
- [C71.3 — The visual build](03-visual-build.md): kit → wheels → paint → livery, as a walkthrough.
- [C71.4 — Modding a car's files](04-modding-files.md): which `BIN`s hold what, and the safe (size-neutral) editing
  pass.
- [C71.5 — The complete car](05-complete-car.md): how all systems compose, from `CarType` to a driven, moddable
  car.

---

## 71.1 Anatomy of a car

A car is **one object, two data domains** ([C71.1](01-anatomy.md)): the `PlayerCar`
([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)) holds a *performance config* (installed `PART_*`
parts selecting vault values, [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and a *visual config*
(active `KIT`, paint, vinyls in the file set, [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)).
The sim reads the vault; the renderer reads the file set. Understanding a car is holding both domains at once.

## 71.2 The performance build

Building performance ([C71.2](02-performance-build.md)) is the loop: pick a class
([C69.1](../C69-Performance-Upgrades-Tuning/01-classes-tiers.md)), buy the next tier
([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)), watch the `PERF_` rating
([C69.2](../C69-Performance-Upgrades-Tuning/02-perf-ratings.md)) and the bar
([C69.3](../C69-Performance-Upgrades-Tuning/03-tuning-bars.md)) rise, and feel the sim
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) respond — repeated across the
nine classes to a maxed car.

## 71.3 The visual build

Building appearance ([C71.3](03-visual-build.md)) layers the visual data trinity
([C70.4](../C70-Visual-Customisation/04-vinyls-decals.md)): choose a body kit (geometry,
[C70.1](../C70-Visual-Customisation/01-body-kits.md)), fit wheels/aero
([C70.2](../C70-Visual-Customisation/02-wheels-aero.md)), lay a livery (vinyls baked to `PREVINYL.BIN`,
[C70.4](../C70-Visual-Customisation/04-vinyls-decals.md)), and paint it all
([C70.3](../C70-Visual-Customisation/03-paint.md)) — mesh, texture, colour, composed by the renderer.

## 71.4 Modding a car's files

Modding a car ([C71.4](04-modding-files.md)) means editing its `BIN`s directly: `GEOMETRY.BIN` for meshes
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), the `TPK`s for textures
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), the vault for tuning
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — each with the same size-neutral discipline the world
data needs ([C15.7](../C15-Track-Streaming/07-section-contents.md)), and the full workflow safety of
[Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md).

---

### Key takeaways

- A car is **one object, two data domains** — a performance config (vault,
  [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) read by the sim, and a visual config (file set,
  [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)) read by the renderer — held by the
  `PlayerCar` ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)).
- The **performance build** is a loop — class → tier → rating → bar → sim
  ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)) — across nine classes to a
  maxed car.
- The **visual build** layers the trinity — geometry (kit) + colour (paint) + texture (livery),
  [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md) — composed by the renderer.
- **Modding** edits the `BIN`s directly (`GEOMETRY.BIN`, the `TPK`s, the vault) with **size-neutral discipline** and
  the workflow safety of [Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md).
- This chapter is the **synthesis** — every claim traces to a verified source chapter; it's the map of how the whole
  car story fits together.

**Next:** [C71.1 — Anatomy of a car](01-anatomy.md).
