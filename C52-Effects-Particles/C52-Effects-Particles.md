# Chapter 52 — Effects, Particles & the FX Bank

> **Goal of this chapter:** decode the two effect worlds — the pooled **particle system** (`Emitter*`/`Particle*`
> pools, `ParticlesShader`) that makes the smoke, sparks, dust and debris, and the **screen post-process effects**
> (`EffectRadialBlur`/`EffectVignette`/`EffectBrightness`) that compose MW's look — plus the per-entity `Effects*`
> classes and the surface×mode FX catalogue (`fxtd_*`/`fxcar_*`).

The rendered scene ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) is only half the picture — over
and through it play the *effects*: tyre smoke on a drift, sparks off a wall scrape, dust on a dirt road, the
debris of a smashed prop, the speed-blur of nitrous. This chapter decodes the effect systems: the particle
emitters that spawn those visuals, the effect catalogue that maps a situation to the right effect, and the
per-entity effect classes that attach them to cars, fragments, and smackables.

> **Verified against the executable.** The particle system is **pooled**: `EmitterSlotPool`, `EmitterGroupSlotPool`,
> `EmitterPositionSlotPool`, and `ParticleSlotPool` are pre-sized allocators; `ParticlesShader` and
> `ParticleSystemEnable` drive rendering. The **screen post-process effects** — `EffectBrightness`,
> `EffectRadialBlur`, `EffectVignette`, `EffectTarget` — are the composable components of the visual treatment
> ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)). The **per-entity effect classes** — `EffectsCar`,
> `EffectsVehicle`, `EffectsFragment`, `EffectsSmackable`, `EffectsPlayer` — attach effects via `EffectConn`. The
> surface×mode FX (`fxtd_*` terrain-drive, `fxcar_*` car impacts) are catalogued in the effect bank / ErtS data.

---

## Deep-dive pages

- [C52.1 — The two effect worlds](01-two-worlds.md): scene particles vs. screen post-process.
- [C52.2 — Emitters & particles](02-emitters-particles.md): the pooled particle system.
- [C52.3 — The FX catalogue](03-fx-catalogue.md): the `fxtd_*`/`fxcar_*` surface×mode effects.
- [C52.4 — Per-entity effects](04-entity-effects.md): `Effects*` classes and `EffectConn`.
- [C52.5 — Reading effects in RE](05-reading-effects.md): navigating the effect systems.

---

## 52.1 Two effect worlds

There are **two distinct effect systems** ([C52.1](01-two-worlds.md)): **scene particles** (3D — smoke, sparks,
dust, spawned by emitters into the world) and **screen post-process** (2D — brightness, radial blur, vignette,
applied to the whole frame). They're different in kind — one adds *things in the world*, the other *filters the
image* — and MW uses both. The post-process components (`EffectRadialBlur`, `EffectVignette`, `EffectBrightness`)
are the building blocks of the `VisualTreatment` ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).

## 52.2 Emitters & particles

The scene particles come from a **pooled emitter/particle system** ([C52.2](02-emitters-particles.md)): an
**emitter** (from `EmitterSlotPool`/`EmitterGroupSlotPool`) spawns **particles** (from `ParticleSlotPool`) at a
position (`EmitterPositionSlotPool`), which are drawn by `ParticlesShader`. Everything is **pooled**
([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — fixed slots, no per-frame heap churn — because
effects spawn and die constantly (every drift, every impact). `ParticleSystemEnable` gates the whole system.

## 52.3 The FX catalogue

Which effect plays in a situation is a **catalogue lookup** ([C52.3](03-fx-catalogue.md)): the `fxtd_*` (terrain-drive)
effects keyed by *surface × tyre mode* ([Chapter 44](../C44-Surfaces-Grip/04-tire-effects.md)) — `fxtd_sl_asphalt`
(drift smoke on asphalt), `fxtd_dr_sand` (dust on sand) — and the `fxcar_*` effects keyed by *impact type*
([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) — sparks off a wall, a crunch off a car. The
catalogue (the effect bank / ErtS table) is the grid that maps every driving/collision situation to its effect.

## 52.4 Per-entity effects

Effects attach to entities through the **`Effects*` classes** ([C52.4](04-entity-effects.md)): `EffectsCar`/
`EffectsVehicle` (a car's effects — exhaust, tyre smoke, damage sparks), `EffectsFragment` (a broken-off part's
effects), `EffectsSmackable` (a knocked prop's debris,
[Chapter 43](../C43-Collision-Contacts/05-smackables.md)), `EffectsPlayer`. Each is fed by an **`EffectConn`**
connector ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) — the entity publishes its state, the effects class
spawns the matching particles. So a car's effects are a mechanic-like component
([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) reading the sim and emitting visuals.

---

### Key takeaways

- There are **two effect worlds**: **scene particles** (3D smoke/sparks/dust from emitters) and **screen
  post-process** (2D brightness/blur/vignette) — the latter composing the `VisualTreatment`.
- The particle system is **pooled** — `EmitterSlotPool`/`EmitterGroupSlotPool`/`ParticleSlotPool` +
  `ParticlesShader` — no per-frame heap churn (effects spawn/die constantly).
- The **FX catalogue** maps situations to effects — `fxtd_*` by surface×tyre-mode
  ([Chapter 44](../C44-Surfaces-Grip/04-tire-effects.md)), `fxcar_*` by impact type
  ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)).
- **Per-entity `Effects*` classes** (`EffectsCar`/`EffectsVehicle`/`EffectsFragment`/`EffectsSmackable`) attach
  effects via **`EffectConn`**, reading sim state and emitting visuals.
- The post-process components (`EffectRadialBlur`/`EffectVignette`/`EffectBrightness`) are the **building blocks of
  MW's look** ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).

**Next:** [Chapter 53 — Cameras & the Director](../C53-Cameras-Director/C53-Cameras-Director.md): how the game
frames the action.
