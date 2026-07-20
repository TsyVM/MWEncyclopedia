# C40.6 — DAMAGE, DRAW & SOUND: Consequence & Presentation

> **The one-sentence version:** three mechanics *read* the sim's state rather than drive it —
> `BEHAVIOR_MECHANIC_DAMAGE` turns collisions into deformation and performance loss, `BEHAVIOR_MECHANIC_DRAW`
> renders the car, `BEHAVIOR_MECHANIC_SOUND` produces its audio.

[← C40.5 — SUSPENSION](05-suspension.md) · [Chapter 40 hub](C40-Eight-Mechanics.md) ·
[Next: C40.7 — Reading the mechanics in RE →](07-reading-mechanics.md)

---

## The output mechanics

The first five mechanics ([C40.2](02-rigidbody.md)–[C40.5](05-suspension.md)) *drive* the simulation — they
produce forces and motion. The last three — DAMAGE, DRAW, SOUND — *read* the simulation and express it. They're
the **output mechanics**: they don't change where the car goes, they turn where it goes (and what it hits) into
deformation, pixels, and audio. This is a clean split — inputs and physics on one side, consequences and
presentation on the other ([C40.1](01-the-mechanic-model.md)).

## DAMAGE: collisions become consequence

`BEHAVIOR_MECHANIC_DAMAGE` reads the car's **collisions** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md))
and turns them into damage ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)):

- **Deformation.** A collision's impulse, above a threshold, deforms the affected zone — the mesh crumples, parts
  detach ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)).
- **Performance loss.** Enough damage degrades performance — a damaged engine loses power, a damaged suspension
  handles worse — feeding back into the other mechanics.
- **Damage zones.** The car is divided into zones (front, rear, sides), each accumulating damage independently
  ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)).

The string table places `DamageVehicle` immediately after the mechanic list
([C40.1](01-the-mechanic-model.md)) — the damage schema follows the mechanic schema, consistent with damage being
a first-class vehicle behaviour. So DAMAGE is the mechanic that gives collisions *weight* — a crash isn't just a
bump, it dents the car and costs performance.

> ✅ *Verified:* `BEHAVIOR_MECHANIC_DAMAGE` is present in `attributes.bin`, immediately followed in the string
> region by `DamageVehicle` ([C40.1](01-the-mechanic-model.md)) — the damage schema adjacent to the mechanic
> table. Damage zones and deformation are detailed in
> [Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md).

## DRAW: the car's appearance

`BEHAVIOR_MECHANIC_DRAW` is the car's **visual representation** — the mechanic that renders the vehicle each
frame at its simulated transform ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)):

- **The model.** The car's geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) and
  textures ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) — the mesh drawn at the body's position.
- **Wheels and motion.** The wheels spin and steer to match the sim (wheel rotation from speed, steer angle from
  the controls) — visual state driven by physical state.
- **Damage state.** The draw reflects the DAMAGE mechanic's deformation — the crumpled mesh, missing parts
  ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)).

So DRAW is the mechanic that makes the car *visible* — reading the sim's transform and state and turning them into
the rendered vehicle. It's presentation: it consumes the sim, it doesn't affect it.

## SOUND: the car's voice

`BEHAVIOR_MECHANIC_SOUND` produces the car's **audio** — the mechanic that reads the sim and drives the sound
engine ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)):

- **Engine sound.** The GIN synth ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) driven by the
  engine mechanic's RPM ([C40.4](04-engine.md)) — the revving, shifting, and boost.
- **Tyre sound.** Screech and roll noise from the tyre slip and surface
  ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) — the RoadNoise/TireEffect records.
- **Collision sound.** Impacts from the DAMAGE mechanic's collisions — crunches and scrapes.

So SOUND is the car's voice — it reads the engine RPM, the tyre slip, and the collisions, and turns them into the
audio that makes the car feel alive. Like DRAW, it consumes the sim; unlike the driving mechanics, it produces
no force.

## Why separate output from physics

Splitting the readers (DAMAGE/DRAW/SOUND) from the drivers (the first five) is the same isolation the connectors
provide ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)):

- **The physics stays pure.** DRAW and SOUND read the sim's published state; they can't perturb it. The sim is
  deterministic regardless of whether anyone's watching or listening.
- **Presentation can vary independently.** The rendering and audio can be detailed or simple (LOD, culling)
  without changing the physics — a distant car simulates the same but draws/sounds cheaper.
- **DAMAGE bridges.** DAMAGE is the one output mechanic that *does* feed back (performance loss), but it does so
  through the same parameter channels the other mechanics read — a clean, bounded feedback, not a tangle.

So the three output mechanics complete the car: the drivers make it move, the outputs make it *matter* (damage),
*visible* (draw), and *audible* (sound). Together with the five drivers, they're the whole vehicle
([C40.7](07-reading-mechanics.md)).

## RE implications

- **DAMAGE, DRAW, SOUND are the output mechanics** — they read the sim, they don't drive it.
- **DAMAGE** turns collisions ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) into
  deformation + performance loss ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)); `DamageVehicle`
  is adjacent in the string table.
- **DRAW** renders the car at its transform; **SOUND** drives the audio (engine RPM, tyre slip, collisions,
  [Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).
- **Separating output from physics** keeps the sim pure and lets presentation vary (LOD) independently.

---

### Key takeaways

- Three mechanics **read** the sim rather than drive it: `DAMAGE`, `DRAW`, `SOUND` — the output mechanics.
- **DAMAGE** turns collisions into deformation and performance loss (zones,
  [Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)); `DamageVehicle` sits right after the
  mechanic table in `attributes.bin`.
- **DRAW** renders the car at its simulated transform (model, spinning/steering wheels, damage state).
- **SOUND** is the car's voice — engine RPM ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)), tyre
  slip, collisions.
- Separating outputs from physics keeps the sim **pure and deterministic**, with presentation free to LOD; DAMAGE
  is the one bounded feedback.

**Continue:** [C40.7 — Reading the mechanics in RE](07-reading-mechanics.md) · [Chapter 40 hub](C40-Eight-Mechanics.md)
