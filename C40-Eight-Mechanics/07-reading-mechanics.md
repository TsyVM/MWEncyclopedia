# C40.7 — Reading the Mechanics in RE

> **The one-sentence version:** find the mechanic set from the `BEHAVIOR_MECHANIC_*` string table in
> `attributes.bin`, read each mechanic's parameters from the vault, and understand a car as the composition of
> eight tunable, swappable behaviours.

[← C40.6 — DAMAGE, DRAW & SOUND](06-damage-draw-sound.md) · [Chapter 40 hub](C40-Eight-Mechanics.md) ·
[Next: Chapter 41 — Physics & Rigid-Body Dynamics →](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)

---

## Anchors for mechanic RE

The mechanic set is one of the most legible parts of the vehicle system, because it's named in the data:

- **The string table** in `attributes.bin` (offsets 17110–17285) — the eight `BEHAVIOR_MECHANIC_*` names
  ([C40.1](01-the-mechanic-model.md)).
- **The vault** ([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)–[13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
  — each mechanic's parameters, keyed by reflection-hashed field names.
- **The sim pipeline** ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) — where the mechanics
  run: `Physics::Simulate (0x6BB4D0)`.
- **`DamageVehicle`** (adjacent to the table) — the entry into the damage schema
  ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)).

From these, a car's behaviour is fully mapped: the eight mechanics, their parameters, and where they execute.

## The RE workflow

Reading a car's mechanics:

1. **Find the string table** — grep `attributes.bin` for `BEHAVIOR_MECHANIC_` to confirm the eight names and
   their layout ([C40.1](01-the-mechanic-model.md)).
2. **Map each mechanic to its parameters** — the vault categories
   ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) hold the per-car values: engine (power,
   gears), suspension (springs), damage (thresholds).
3. **Locate where they run** — `Physics::Simulate` ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
   drives the driving mechanics; the render and audio subsystems drive DRAW and SOUND.
4. **Compare cars** — read two cars' parameter sets to see how they differ (same mechanics, different values).

The output is a complete picture of a car: eight behaviours, each with its parameters, composed into the vehicle.

## Composition is the insight

The single most important thing to carry away from this chapter is that **a car is composition**
([C40.1](01-the-mechanic-model.md)):

- **One codebase, many cars.** The eight mechanics' *code* is shared across every car; only the *parameters*
  differ. The garage is data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), not code.
- **Player and world unified.** Swapping the driver mechanic (INPUT ↔ AI, [C40.3](03-ai-and-input.md)) turns a
  player car into a cop or traffic car — same physics, different driver. Everything on the road is the same kind
  of object.
- **Behaviours are independent and ordered.** Each mechanic is a separable piece
  ([C40.1](01-the-mechanic-model.md)), run in a dependency order (driver → engine → suspension → integrate →
  output). You can reason about one mechanic without the others.

This is the architecture that makes Most Wanted's driving both *deep* (eight interacting behaviours) and
*scalable* (dozens of cars, cops, and traffic from one physics core). Understanding the eight mechanics *is*
understanding how the game models a car.

## Verifying a mechanic claim

To verify a claim about the mechanics, reduce it to a check:

- **"Are there eight mechanics?"** — grep `attributes.bin`: exactly eight `BEHAVIOR_MECHANIC_*`, once each
  ([C40.1](01-the-mechanic-model.md)). ✅
- **"Is X a mechanic?"** — is `BEHAVIOR_MECHANIC_X` in the table? (AI/INPUT/ENGINE/SUSPENSION/RIGIDBODY/DAMAGE/
  DRAW/SOUND yes; anything else no.)
- **"Where does the engine's RPM go?"** — to the GIN synth
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md), verified `Gnsu` rpm range).
- **"What tunes a car?"** — the vault parameters per mechanic
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).

Each reduces to a grep, an offset, or a cross-reference — the verification-first discipline
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)).

## RE implications

- **Anchor on the `BEHAVIOR_MECHANIC_*` string table** (`attributes.bin` 17110–17285) and the vault parameters.
- **Map each mechanic to its parameters** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and its
  run site ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).
- **Composition is the insight** — one codebase, many cars; player and world unified by the swappable driver.
- **Every claim reduces to a check** — the eight names, the parameters, the run sites.

---

### Key takeaways

- The mechanic set is **named in the data** — grep `attributes.bin` for the eight `BEHAVIOR_MECHANIC_*` and map
  each to its **vault parameters**.
- The RE workflow: find the table → map parameters → locate run sites (`Physics::Simulate`) → compare cars.
- **Composition is the core insight** — one shared codebase, many cars by parameters; player/cop/traffic unified
  by the swappable driver mechanic.
- The eight mechanics are **independent and ordered** — reason about one at a time, in the pipeline order.
- Understanding the eight mechanics **is** understanding how Most Wanted models a car.

**Next:** [Chapter 41 — Physics & Rigid-Body Dynamics](../C41-Physics-RigidBody/C41-Physics-RigidBody.md): the
rigid-body class hierarchy the RIGIDBODY mechanic is built on.

**Sources:** `GLOBAL/attributes.bin` (verified: eight `BEHAVIOR_MECHANIC_*` strings, contiguous, offsets
17110–17285); `speed.exe` (verified: `Physics::Simulate 0x6BB4D0`, `IntegrateMotion 0x6BA510`,
`Physics_Base::ctor 0x6B9920`).
