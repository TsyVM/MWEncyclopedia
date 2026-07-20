# C40.1 — The Mechanic Model

> **The one-sentence version:** a car's behaviour is a set of named components — the eight `BEHAVIOR_MECHANIC_*`,
> stored as a contiguous string table in `attributes.bin` — each with vault-tuned parameters and a per-frame
> update, composed into one vehicle.

[← Chapter 40 hub](C40-Eight-Mechanics.md) · [Next: C40.2 — RIGIDBODY →](02-rigidbody.md)

---

## The verified string table

The eight mechanics are not a documentation invention — they are literally named in the game's attribute data.
In `GLOBAL/attributes.bin`, the eight `BEHAVIOR_MECHANIC_*` strings sit as a **contiguous, null-separated string
table**:

```
17110: BEHAVIOR_MECHANIC_RIGIDBODY
17138: BEHAVIOR_MECHANIC_AI
17159: BEHAVIOR_MECHANIC_INPUT
17183: BEHAVIOR_MECHANIC_ENGINE
17208: BEHAVIOR_MECHANIC_SUSPENSION
17237: BEHAVIOR_MECHANIC_DAMAGE
17262: BEHAVIOR_MECHANIC_DRAW
17285: BEHAVIOR_MECHANIC_SOUND
```

They appear **exactly once each**, packed back-to-back (each string's null terminator immediately precedes the
next), immediately after the part name `k500_hood` and immediately before `DamageVehicle`. That layout — a tight
run of eight enum-like strings inside the vehicle schema region — is the signature of an **enumerated set**: the
schema's declaration of "these are the eight behaviours a vehicle has."

> ✅ *Verified:* the eight `BEHAVIOR_MECHANIC_*` strings are present exactly once each in
> `GLOBAL/attributes.bin`, contiguous and null-separated, at offsets 17110–17285, in the order RIGIDBODY, AI,
> INPUT, ENGINE, SUSPENSION, DAMAGE, DRAW, SOUND.

## What a mechanic is

A **mechanic** is a behaviour component — one facet of a car's behaviour, encapsulated:

- **A name** — `BEHAVIOR_MECHANIC_<X>` — its identity in the schema and (as a reflection-hashed key,
  [Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) its lookup handle.
- **Parameters** — a set of tuning values in the vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
  — the engine's power curve, the suspension's stiffness, the damage thresholds — that define *this car's* version
  of the behaviour.
- **A per-frame update** — the code that runs the behaviour each tick, reading the parameters and the car's
  state, producing forces or effects.

So a mechanic bundles *what a behaviour is* (the update code, shared across cars) with *how this car does it* (the
parameters, per car). A car is eight such bundles.

## Composition, not inheritance

The key design choice is that a car is built by **composition** — it *has* eight mechanics — rather than by a
deep inheritance hierarchy. This is the classic components-over-inheritance pattern:

- **A car is a bag of components.** Each mechanic is a separable piece; the car is their aggregation. Adding a
  behaviour means adding a mechanic, not editing a monolithic vehicle class.
- **Behaviours are independent.** The engine mechanic doesn't know about the sound mechanic; they communicate
  through the shared body state (the rigid body, [C40.2](02-rigidbody.md)), not directly. This keeps each
  mechanic simple and testable.
- **Parameters drive variation.** Two cars differ by their mechanics' *parameters*, not their *code*. The garage
  is data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).

So the mechanic model is Most Wanted's answer to "how do you build many different cars without many different
codebases": one set of eight behaviours, parameterised per car, composed into each vehicle.

## The order matters

The eight run in an **order** each frame (driven by `Physics::Simulate`,
[Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), and the order is a dependency chain:

```
driver (AI/INPUT) → engine → suspension → rigidbody integrate → damage/draw/sound (read the result)
```

The driver decides, the engine powers, the suspension loads the tyres, the rigid body integrates, and the output
mechanics express the result. Running them out of order would break the chain (the engine can't use controls that
haven't been read yet). So the mechanic set is not just a bag — it's an *ordered* pipeline
([C39.6](../C39-Vehicle-Simulation/06-input-to-tyres.md)), and the file-table order (RIGIDBODY first) reflects
the body being the substrate the rest act on.

> 🟡 *Reasoned:* the per-frame execution order of the mechanics (driver → power → grip → integrate → output) is
> the standard vehicle-component pipeline, consistent with the verified sim chain
> ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)); the exact scheduling within `Simulate` is
> deeper RE. The *set* of eight and their names are verified.

## RE implications

- **The eight mechanics are a verified string table** in `attributes.bin` — an enumerated set, not a guess.
- **A mechanic = name + vault parameters + per-frame update** — a behaviour component.
- **A car is composition** — a bag of eight mechanics, varied by parameters not code.
- **The mechanics run in an ordered pipeline** — driver → engine → suspension → integrate → output.

---

### Key takeaways

- The eight `BEHAVIOR_MECHANIC_*` are a **verified contiguous string table** in `attributes.bin` (offsets
  17110–17285) — the schema's enumerated set of vehicle behaviours.
- A **mechanic** bundles a name, vault-tuned parameters, and a per-frame update — one facet of a car's behaviour.
- A car is built by **composition** (a bag of eight components), varied car-to-car by **parameters, not code**.
- The mechanics run in an **ordered pipeline** each frame — driver → engine → suspension → integrate → output.
- This model is how Most Wanted ships many distinct cars on one shared physics engine.

**Continue:** [C40.2 — RIGIDBODY: the physical body](02-rigidbody.md) · [Chapter 40 hub](C40-Eight-Mechanics.md)
