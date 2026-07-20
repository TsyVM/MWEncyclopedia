# C40.2 — RIGIDBODY: The Physical Body

> **The one-sentence version:** `BEHAVIOR_MECHANIC_RIGIDBODY` is the car's physical body — the mass, velocity,
> and transform that the other seven mechanics act on, integrated each frame by `IntegrateMotion` (`0x6BA510`).

[← C40.1 — The mechanic model](01-the-mechanic-model.md) · [Chapter 40 hub](C40-Eight-Mechanics.md) ·
[Next: C40.3 — AI & INPUT →](03-ai-and-input.md)

---

## The substrate mechanic

`BEHAVIOR_MECHANIC_RIGIDBODY` is first in the string table ([C40.1](01-the-mechanic-model.md)) for a reason: it's
the **substrate** the other mechanics act on. It owns the car's core physical state:

- **Mass and inertia** — how heavy the car is and how it resists rotation.
- **Velocity** (linear and angular) — how the car is moving.
- **The transform** — where the car is (position + orientation), at `[this+0xF0]` on the body
  ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)).
- **Accumulated force and torque** — the sum of what the other mechanics push with this frame.

Every other mechanic ultimately expresses itself as a force or torque *on the rigid body*: the engine's drive
force, the suspension's normal force, the tyres' grip forces, a collision's impulse. The rigid body accumulates
them and integrates ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)).

## It runs the integrate

The rigid-body mechanic is what runs `IntegrateMotion` (`0x6BA510`,
[C39.4](../C39-Vehicle-Simulation/04-integrate.md)) — turning the accumulated force/torque into a change in
velocity and position over the timestep. So the rigid body is both the *state* (mass, velocity, transform) and
the *integrate step* that advances it. The other mechanics contribute forces; the rigid body is where those
forces become motion.

This is why the sim pipeline ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) ends in the
rigid body's integrate: the frame's work is *accumulate forces (all mechanics) → integrate (rigid body)*. The
rigid body is the convergence point.

> ✅ *Verified:* the rigid-body integrate is `IntegrateMotion (0x6BA510)`, opening `sub esp, 0x530`
> ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)); the body's transform is at `[this+0xF0]` and its part
> array at `[this+0xEC]` ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)). The class hierarchy of rigid
> bodies (`RigidBody`, `SimpleRigidBody`, `RBVehicle`, `RBCop`) is verified in
> [Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md).

## Built on Physics_Base

The rigid body is built on **`Physics_Base`** — whose constructor at `0x6B9920` embeds **three interface
sub-objects** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)). A `RigidBody` is a
`Physics_Base` with these interfaces (the physics body implements multiple interfaces — the simulate interface,
the collision interface, and more), so it can be treated as any of them by the systems that need one. This
multi-interface design ([C41.2](../C41-Physics-RigidBody/02-physics-base.md)) is how one body plugs into the sim,
the collision world, and the mechanics simultaneously.

> ✅ *Verified:* `Physics_Base::ctor` at `0x6B9920` begins `6A FF 68 12 DF 87 00` (SEH prologue) and embeds three
> interface sub-objects ([Chapter 41](../C41-Physics-RigidBody/02-physics-base.md)).

## Why the body is a mechanic

It might seem odd to call the rigid body a "mechanic" alongside the engine and suspension — isn't it *the car*
rather than a behaviour of it? But treating it as a mechanic is consistent and useful:

- **Uniformity.** Every facet of the car, including its physical presence, is a mechanic — one model, no special
  cases. The composition ([C40.1](01-the-mechanic-model.md)) is total.
- **Tunable.** The rigid body has parameters too — mass, centre of gravity, inertia
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — tuned per car like any mechanic. A heavier
  car is a different rigid-body parameter set.
- **Swappable substrate.** Different body types ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md))
  — a full vehicle vs. a simple prop — are different rigid-body implementations under the same mechanic slot.

So the rigid body is the mechanic that provides the *physical object* the others decorate with behaviour — the
foundation of the eight, and the one they all ultimately push on.

## RE implications

- **`BEHAVIOR_MECHANIC_RIGIDBODY`** is the substrate — mass, velocity, transform, accumulated force; the other
  mechanics act on it.
- **It runs the integrate** (`IntegrateMotion 0x6BA510`) — where accumulated forces become motion.
- **It's built on `Physics_Base`** (ctor `0x6B9920`, three interfaces) — one body, many interfaces.
- **It's a mechanic like the others** — uniform composition, tunable (mass/CoG/inertia), swappable substrate.

---

### Key takeaways

- `BEHAVIOR_MECHANIC_RIGIDBODY` is the **physical body** — the mass, velocity, transform (`[this+0xF0]`), and
  accumulated force the other seven mechanics act on.
- It runs the **integrate** (`IntegrateMotion 0x6BA510`) — the convergence point where all mechanics' forces
  become motion.
- It's built on **`Physics_Base`** (ctor `0x6B9920`) with **three interface sub-objects** — one body usable as
  many interfaces.
- Treating the body as a **mechanic** keeps the composition uniform: it's tunable (mass/CoG) and swappable
  (vehicle vs. prop) like any other.
- The rigid body is the **foundation** of the eight — the physical object the rest decorate.

**Continue:** [C40.3 — AI & INPUT: the two drivers](03-ai-and-input.md) · [Chapter 40 hub](C40-Eight-Mechanics.md)
