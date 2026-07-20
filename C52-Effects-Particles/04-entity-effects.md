# C52.4 — Per-Entity Effects

> **The one-sentence version:** effects attach to entities through the `Effects*` classes — `EffectsCar`/
> `EffectsVehicle` (a car's exhaust, smoke, damage sparks), `EffectsFragment` (a broken part), `EffectsSmackable`
> (a knocked prop), `EffectsPlayer` — each fed by an `EffectConn` connector reading the entity's sim state.

[← C52.3 — The FX catalogue](03-fx-catalogue.md) · [Chapter 52 hub](C52-Effects-Particles.md) ·
[Next: C52.5 — Reading effects in RE →](05-reading-effects.md)

---

## Effects as an entity component

An effect doesn't float free — it belongs to an *entity* (a car, a broken part, a prop), and the **`Effects*`
classes** are how effects attach. Each is the *effect component* of its entity type
([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)):

| Class | Entity | Effects |
|---|---|---|
| `EffectsCar` / `EffectsVehicle` | a car | exhaust, tyre smoke, damage sparks, nitrous flame |
| `EffectsFragment` | a broken-off part ([Chapter 45](../C45-Damage-Deformation/03-deformation.md)) | debris trail, sparks |
| `EffectsSmackable` | a knocked prop ([Chapter 43](../C43-Collision-Contacts/05-smackables.md)) | scatter debris, dust |
| `EffectsPlayer` | the player specifically | player-only effect flourishes |

So each entity that produces visuals has an `Effects*` component that knows *which* effects that entity spawns and
*when*. A car's `EffectsVehicle` knows to emit tyre smoke when drifting, sparks when scraping, flame when on
nitrous — reading the car's state and spawning the matching catalogue effects ([C52.3](03-fx-catalogue.md)).

> ✅ *Verified:* `EffectsCar`, `EffectsVehicle`, `EffectsFragment`, `EffectsSmackable`, `EffectsPlayer`, and the
> connector `EffectConn` are present in `speed.exe`; `EffectsVehicle` is a vault key
> ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)). These are the per-entity effect classes.

## EffectConn: sim state to effects

The `Effects*` component reads its entity's state through an **`EffectConn`** connector
([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) — the same one-way boundary as the render connector
([C51.2](../C51-Render-Pipeline/02-render-objects.md)):

- **The entity publishes state** — the car's speed, tyre slip/mode ([C44.4](../C44-Surfaces-Grip/04-tire-effects.md)),
  surface, damage ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)), nitrous state.
- **The `Effects*` component reads it** — via `EffectConn` — and decides which effects to spawn.
- **The effects spawn** — the matching emitter groups ([C52.2](02-emitters-particles.md)) from the catalogue
  ([C52.3](03-fx-catalogue.md)), at the right positions (the wheels, the exhaust, the impact points).

So the flow is *entity state → `EffectConn` → `Effects*` component → spawn catalogue effect → particles*. This makes
effects a *reader* of the simulation, like the DRAW and SOUND mechanics
([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) — it consumes sim state and produces presentation, never
perturbing the sim. A car's effects are the visual echo of its physics: drift → smoke, scrape → sparks, crash →
debris, all driven by the state the car publishes.

## The car's effects, in detail

`EffectsVehicle`/`EffectsCar` is the richest per-entity effects component — a car produces many effects, each tied
to a state:

- **Tyre smoke** — when the tyre slips (drift/skid, [C44.4](../C44-Surfaces-Grip/04-tire-effects.md)), the
  surface×mode effect ([C52.3](03-fx-catalogue.md)) spawns at the wheel.
- **Exhaust** — a steady emitter at the tailpipe, intensity tracking throttle/RPM
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)).
- **Nitrous flame** — when NOS fires ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)), a
  flame/heat effect from the exhaust.
- **Damage sparks/smoke** — as the car takes damage ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)),
  sparks from scrapes and smoke from a wrecked engine.

So the car's effects are a *bundle* of state-driven emitters, each reading a different aspect of the car's sim
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) and producing the matching visual. This is why a
car *looks* like it's doing what it's doing — the smoke, flame, sparks, and exhaust are all `EffectsVehicle`
reading the physics and spawning the right particles. The effects component is the car's visual voice, as the sound
mechanic ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) is its audible one.

## Why per-entity effect classes

Giving each entity type its own `Effects*` class (rather than a generic effect spawner) fits the composition model
([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)):

- **Each entity knows its own effects.** A car's effects (`EffectsVehicle`) differ from a prop's
  (`EffectsSmackable`) — different states, different visuals. Encapsulating each in its own class keeps the logic
  where it belongs.
- **They're components, tuned by data.** `EffectsVehicle` is a vault-keyed spec
  ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)) — a car's effect parameters (which effects, how much)
  are data, like its other mechanics ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).
- **They compose the existing systems** — the effects classes read the sim (via `EffectConn`) and spawn from the
  catalogue ([C52.3](03-fx-catalogue.md)) using the particle pools ([C52.2](02-emitters-particles.md)) — no new
  machinery, just composition.

So the per-entity effects classes are the *glue* between the simulation and the particle system: each entity's
`Effects*` component reads its state and spawns its catalogue effects. They're the reason effects are *attached to
the right things doing the right things* — the car smokes, the prop scatters, the fragment trails — each entity's
own visual behaviour, composed from the shared effect machinery.

## RE implications

- **`Effects*` classes** attach effects to entities — `EffectsCar`/`EffectsVehicle`, `EffectsFragment`,
  `EffectsSmackable`, `EffectsPlayer`.
- **`EffectConn`** feeds sim state to the effects component (one-way) — effects *read* the sim, like DRAW/SOUND.
- **A car's effects** are a bundle of state-driven emitters — tyre smoke, exhaust, nitrous, damage sparks.
- **Per-entity classes** encapsulate each entity's effects, are vault-tuned, and compose the catalogue + pools.

---

### Key takeaways

- Effects attach to entities through **`Effects*` classes** — `EffectsCar`/`EffectsVehicle`, `EffectsFragment`,
  `EffectsSmackable`, `EffectsPlayer` — each its entity's **effect component**.
- The **`EffectConn` connector** feeds entity state to the effects component (one-way boundary) — effects **read**
  the sim, like the DRAW and SOUND mechanics, never perturbing it.
- A **car's effects** (`EffectsVehicle`) bundle many state-driven emitters — tyre smoke (drift), exhaust
  (throttle), nitrous flame, damage sparks — the visual echo of its physics.
- The flow is **entity state → `EffectConn` → `Effects*` → catalogue lookup → particle spawn**.
- Per-entity classes are **vault-tuned components** that compose the catalogue ([C52.3](03-fx-catalogue.md)) and
  pools ([C52.2](02-emitters-particles.md)) — the glue between simulation and particles.

**Continue:** [C52.5 — Reading effects in RE](05-reading-effects.md) · [Chapter 52 hub](C52-Effects-Particles.md)
