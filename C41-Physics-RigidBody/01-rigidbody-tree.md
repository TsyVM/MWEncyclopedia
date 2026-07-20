# C41.1 — The RigidBody Class Tree

> **The one-sentence version:** the physics bodies form a verified class tree rooted at `RigidBody` — with
> `SimpleRigidBody`, `RBSmackable`, `RBVehicle` (→ `RBCop`), and `RBTrailer` as specialisations — all present as
> strings in `speed.exe`'s `.rdata`.

[← Chapter 41 hub](C41-Physics-RigidBody.md) · [Next: C41.2 — Physics_Base →](02-physics-base.md)

---

## The verified strings

The rigid-body classes are not inferred — they're literally named in the executable. In `speed.exe`'s `.rdata`,
the class names appear in two clusters:

```
0x4AA4CC  SimpleRigidBody
0x4AAA04  RBSmackable
0x4AAA10  RigidBody
...
0x4ADD54  RBVehicle
0x4ADD60  RBTrailer
0x4ADD6C  RBCop
```

These are null-terminated C strings, packed with related names ([C41.6](06-vehicle-types.md)). Their presence is
the ground truth: Most Wanted *has* these physics classes, named exactly so. Everything else in this chapter
builds on this verified set.

> ✅ *Verified:* `SimpleRigidBody`, `RBSmackable`, `RigidBody`, `RBVehicle`, `RBTrailer`, and `RBCop` are all
> present as null-terminated strings in `speed.exe` at the offsets above.

## The hierarchy

The names encode a hierarchy — `RB*` = "RigidBody-derived," and the specialisations refine the base:

```
RigidBody                       — the base physics body
├─ SimpleRigidBody              — a lightweight body (props, debris)
├─ RBSmackable                  — a "smackable" world object (knock-over props)
├─ RBVehicle                    — a full vehicle (player + AI cars)
│    └─ RBCop                   — a cop vehicle (RBVehicle + pursuit)
└─ RBTrailer                    — a towed trailer body
```

- **`RigidBody`** is the base ([C41.2](02-physics-base.md)) — it owns mass, velocity, the transform
  (`[this+0xF0]`, [C39.3](../C39-Vehicle-Simulation/03-part-array.md)), and the integrate
  ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)).
- **`SimpleRigidBody`** is a stripped-down body for objects that need physics but not the full vehicle machinery
  — small debris, simple props. Cheaper to simulate.
- **`RBSmackable`** is the class for the world's knock-over objects — cones, signs, fences
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) — bodies that sit inert until "smacked," then
  react.
- **`RBVehicle`** is the full car: the part array of wheels ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)),
  the eight mechanics ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)), the drivetrain.
- **`RBCop`** specialises `RBVehicle` for cops — same physics, plus the pursuit behaviour
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) and cop-specific traits.
- **`RBTrailer`** is a trailer — a body towed behind another, with the coupling constraint.

> 🟡 *Reasoned:* the exact parent/child edges (e.g. `RBCop` deriving from `RBVehicle`) are inferred from the
> naming and role, consistent with the verified class strings and the class system
> ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)); the precise vtable-chain derivation is
> deeper RE. The *set* of classes and their names are verified.

## One tree, many behaviours

The tree is the class system ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) applied to
physics: one base body defines the shared physics (mass, integrate,
[Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), and each derived class refines it. This is why
the sim pipeline is *polymorphic* ([C41.4](04-simulate-thiscall.md)): `Physics::Simulate` calls virtuals on the
body, and each class in the tree provides its own — a `SimpleRigidBody` simulates as a plain body, an `RBVehicle`
simulates with wheels and drivetrain, an `RBCop` adds pursuit. The same `Simulate` runs them all, dispatching to
each class's behaviour.

So the tree buys two things: **shared physics** (every body integrates the same way) and **specialised behaviour**
(each body type does its own thing on top). This is exactly what a class hierarchy is for, and the physics system
uses it to cover everything from a bouncing cone to a pursuing cop with one substrate.

## The player and the cop are cousins

A striking consequence of the tree: the **player's car (`RBVehicle`) and a cop (`RBCop`) are close relatives** —
`RBCop` is `RBVehicle` plus pursuit. They share the wheels, the engine, the suspension, the damage. This is the
class-level counterpart to the mechanic-level insight ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) that a
cop is a player car with the AI driver: at the mechanic level the driver swaps; at the class level `RBCop`
extends `RBVehicle`. Both express the same design — the world's cars and the player's car are the same kind of
thing, specialised where they must differ. Understanding this relatedness is key to understanding pursuit
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)): the cops chasing you are, physically, cars like yours.

## RE implications

- **The rigid-body classes are verified strings** in `speed.exe` — `RigidBody`, `SimpleRigidBody`, `RBSmackable`,
  `RBVehicle`, `RBCop`, `RBTrailer`.
- **They form a tree** rooted at `RigidBody` — shared physics, specialised behaviour per class.
- **The tree makes the sim polymorphic** — one `Simulate` dispatches to each class's virtual
  ([C41.4](04-simulate-thiscall.md)).
- **Player (`RBVehicle`) and cop (`RBCop`) are close relatives** — the class-level unity of the road.

---

### Key takeaways

- The physics bodies are a **verified class tree** — `RigidBody` (base), `SimpleRigidBody`, `RBSmackable`,
  `RBVehicle` → `RBCop`, `RBTrailer` — all named in `speed.exe`.
- `RigidBody` owns the shared physics (mass, velocity, transform, integrate); each derived class **specialises**
  it.
- The tree makes the sim **polymorphic** — `Physics::Simulate` dispatches to each body type's virtual.
- **`RBCop` is `RBVehicle` + pursuit** — player and cop are close relatives, the class-level "same kind of car."
- This one substrate covers everything from a knocked cone (`RBSmackable`) to a pursuing cop (`RBCop`).

**Continue:** [C41.2 — Physics_Base & its interfaces](02-physics-base.md) · [Chapter 41 hub](C41-Physics-RigidBody.md)
