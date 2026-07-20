# C52.5 — Reading Effects in RE

> **The one-sentence version:** navigate the effects by the particle pools (`Emitter*`/`Particle*`), the
> post-process classes (`EffectRadialBlur`/`Vignette`), the FX catalogue naming (`fxtd_*`/`fxcar_*`), and the
> per-entity `Effects*` classes with `EffectConn` — reading the two effect worlds from pools to presentation.

[← C52.4 — Per-entity effects](04-entity-effects.md) · [Chapter 52 hub](C52-Effects-Particles.md) ·
[Next: Chapter 53 — Cameras & the Director →](../C53-Cameras-Director/C53-Cameras-Director.md)

---

## Anchors for effect RE

The effect systems are anchored on verified structures:

- **The particle pools** — `EmitterGroupSlotPool`, `EmitterSlotPool`, `EmitterPositionSlotPool`, `ParticleSlotPool`
  ([C52.2](02-emitters-particles.md)).
- **The particle shader/gate** — `ParticlesShader`, `ParticleSystemEnable` ([C52.2](02-emitters-particles.md)).
- **The post-process classes** — `EffectBrightness`, `EffectRadialBlur`, `EffectVignette`, `EffectTarget`
  ([C52.1](01-two-worlds.md)).
- **The FX catalogue naming** — `fxtd_<mode>_<surface>`, `fxcar_*` ([C52.3](03-fx-catalogue.md)).
- **The per-entity classes** — `EffectsCar`/`EffectsVehicle`/`EffectsFragment`/`EffectsSmackable`, `EffectConn`
  ([C52.4](04-entity-effects.md)).

From these, both effect worlds are navigable: the particle pipeline and the post-process.

## The RE workflow

Reading effects:

1. **Separate the two worlds** — particles (`Emitter*`/`Particle*`) vs. post-process (`Effect*Blur`/`Vignette`)
   ([C52.1](01-two-worlds.md)); they're different systems.
2. **Trace the particle pipeline** — pools → emitters → particles → `ParticlesShader`
   ([C52.2](02-emitters-particles.md)).
3. **Map the catalogue** — the `fxtd_*`/`fxcar_*` naming and the surface×mode grid ([C52.3](03-fx-catalogue.md)).
4. **Follow the per-entity classes** — how `Effects*` reads sim state via `EffectConn` and spawns catalogue effects
   ([C52.4](04-entity-effects.md)).

The output is the full effects picture: particles, post-process, catalogue, and attachment.

## Effects complete the presentation layer

With effects decoded, the **presentation layer** is complete across three chapters
([51](../C51-Render-Pipeline/C51-Render-Pipeline.md)–52):

- **The renderer** ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) — draws the scene (geometry,
  materials, shaders) and the `VisualTreatment`.
- **Scene particles** (this chapter) — the smoke, sparks, dust *in* the world.
- **Screen post-process** ([C52.1](01-two-worlds.md), [C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) — the
  blur, vignette, and grade *over* the image.

Together they turn the simulation's state into the *full* image the player sees — the drawn world, plus the effects
that make it feel alive, plus the treatment that gives it MW's mood. The presentation layer is the *output* half of
the game ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)): it reads the sim (via render and effect
connectors, [C51.2](../C51-Render-Pipeline/02-render-objects.md), [C52.4](04-entity-effects.md)) and produces the
picture, never perturbing the physics. Reading these chapters shows how "what the game *is*" (the sim) becomes "what
the player *sees*" (the frame).

## The connectors tie presentation to sim

A recurring pattern across the presentation layer is the **connector** ([C39.5](../C39-Vehicle-Simulation/05-connectors.md))
— `RenderConn` ([C51.2](../C51-Render-Pipeline/02-render-objects.md)), `EffectConn` ([C52.4](04-entity-effects.md)),
and the sound connectors ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) all follow the *same one-way
boundary*: the sim publishes state, the presentation reads it, the presentation can't reach back. So the whole
presentation layer is a *reader* of the simulation through connectors — a clean, uniform architecture. Recognising
this pattern ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) is the key to the presentation layer: every
visual and audible thing is a connector-fed reader of the physics, which is why the sim stays deterministic and
isolated no matter how elaborate the presentation.

## RE implications

- **Anchor on** the particle pools, the post-process classes, the FX catalogue naming, and the per-entity
  `Effects*` classes.
- **The RE workflow** — separate the two worlds → trace the particle pipeline → map the catalogue → follow the
  per-entity classes.
- **Effects complete the presentation layer** ([51](../C51-Render-Pipeline/C51-Render-Pipeline.md)–52) — drawn
  scene + particles + post-process.
- **Connectors tie presentation to sim** — `RenderConn`/`EffectConn`/sound, all one-way readers.

---

### Key takeaways

- The effect systems are anchored on the **particle pools** (`Emitter*`/`Particle*`), the **post-process classes**
  (`Effect*Blur`/`Vignette`), the **FX catalogue naming** (`fxtd_*`/`fxcar_*`), and the **per-entity `Effects*`
  classes**.
- The RE workflow: **separate the two worlds → trace the particle pipeline → map the catalogue → follow the
  per-entity classes**.
- Effects **complete the presentation layer** (Chapters 51–52) — the drawn scene, the scene particles, and the
  screen post-process together turn sim state into the full image.
- The **connector pattern** (`RenderConn`/`EffectConn`/sound) ties all presentation to the sim as **one-way
  readers** — the physics stays deterministic no matter how elaborate the visuals.
- Reading these chapters shows how **"what the game is" (the sim) becomes "what the player sees" (the frame)**.

**Next:** [Chapter 53 — Cameras & the Director](../C53-Cameras-Director/C53-Cameras-Director.md): how the game frames
the action.

**Sources:** `speed.exe` (verified: particle pools `EmitterSlotPool`/`EmitterGroupSlotPool`/`EmitterPositionSlotPool`/
`ParticleSlotPool`, `ParticlesShader`, `ParticleSystemEnable`; post-process `EffectBrightness`/`EffectRadialBlur`/
`EffectVignette`/`EffectTarget`; per-entity `EffectsCar`/`EffectsVehicle`/`EffectsFragment`/`EffectsSmackable`/
`EffectsPlayer`, `EffectConn`); effect bank / `TireEffectRecord` (×50) for the `fxtd_*` surface×mode catalogue.
