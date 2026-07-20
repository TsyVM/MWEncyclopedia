# C62.5 — Reading Constraints in RE

> **The one-sentence version:** navigate the linked-body physics by `Constraint`/`Joint`/`JointDetached`,
> `RBTrailer` + `SuspensionTrailer` (99 methods), and `JackKnife`/`AIActionJackKnife` — reading it as coupled
> bodies whose difficulty (and drama) shows in the method counts.

[← C62.4 — The jackknife](04-jackknife.md) · [Chapter 62 hub](C62-Constraints-Joints.md) ·
[Next: Chapter 63 — The Collision World & Spatial Partitioning →](../C63-Collision-World/C63-Collision-World.md)

---

## Anchors for constraint RE

The linked-body physics is anchored on verified strings:

- **The constraints** — `Constraint`, `Joint`, `JointDetached` ([C62.1](01-constraints.md)–[C62.2](02-joints-coupling.md)).
- **The trailer** — `RBTrailer` ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) + `SuspensionTrailer` (99
  methods, [C62.3](03-trailers.md)).
- **The jackknife** — `JackKnife`, `AIActionJackKnife` ([C62.4](04-jackknife.md)).

From these, the linked-body physics is navigable: the constraints, the trailer, and the jackknife.

## The RE workflow

Reading constraints:

1. **Find the constraint classes** — `Constraint`/`Joint` ([C62.1](01-constraints.md)); the linkage mechanism.
2. **Trace the coupling** — the hitch joint and `JointDetached` ([C62.2](02-joints-coupling.md)).
3. **Read the trailer** — `RBTrailer` + `SuspensionTrailer` (the 99 methods,
   [C62.3](03-trailers.md)).
4. **Follow the jackknife** — `JackKnife`/`AIActionJackKnife` to the roadblock system
   ([C62.4](04-jackknife.md)).

The output is the full linked-body picture: constraints, coupling, trailer, and jackknife.

## The method count found the hard case

The standout RE lesson of this chapter is that the **method count located the difficulty**
([C50.4](../C50-Verification-Methodology/04-vtable-verification.md)): `SuspensionTrailer` (99 methods) *out-methods*
the hero-car `SuspensionRacer` (45, [C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) — a surprising
result that the *trailer*, not the sports car, has the most complex suspension. Reading the counts
([C50.4](../C50-Verification-Methodology/04-vtable-verification.md)) told us *where the engine spent its effort*,
and it was the trailer — because a swinging multi-axle box is genuinely harder than a car
([C62.3](03-trailers.md)). This is verification-first RE revealing something *non-obvious*: you'd assume the hero
car has the most complex physics, but the bytes say the trailer does. The method count is a *difficulty map*, and
it points at the trailer — a fact you could only know by counting, not by assuming.

## Constraints complete the physics part

With constraints decoded, the *physics* part of the book is complete across the linked-body extension:

- **Single bodies** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) — the `RigidBody` tree, one
  body integrating freely.
- **Wheel forces** ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) — the
  suspension/tyres a body integrates.
- **Contacts** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) — bodies touching the world.
- **Linked bodies** (this chapter) — constraints joining bodies into systems.

So the physics spans *one body* (free), *its forces* (wheels), *its contacts* (collision), and *linked bodies*
(constraints) — the complete rigid-body dynamics of Most Wanted. Constraints are the *last* extension: from single
bodies to *systems* of coupled bodies, enabling trailers and the jackknife. Reading them completes the physics
picture — the sim can now do everything from a lone spinning car to an articulated jackknifing rig. The next
chapter ([63](../C63-Collision-World/C63-Collision-World.md)) goes deeper on the *collision world* — how bodies find
what they touch, the spatial side of the physics.

## RE implications

- **Anchor on** `Constraint`/`Joint`/`JointDetached`, `RBTrailer` + `SuspensionTrailer`, and `JackKnife`.
- **The RE workflow** — constraint classes → coupling → trailer → jackknife.
- **The method count found the hard case** — `SuspensionTrailer` (99) > `SuspensionRacer` (45); the trailer is the
  difficulty.
- **Constraints complete the physics** — single bodies + forces + contacts + linked bodies.

---

### Key takeaways

- The linked-body physics is anchored on **`Constraint`/`Joint`/`JointDetached`**, **`RBTrailer` +
  `SuspensionTrailer`** (99 methods), and **`JackKnife`/`AIActionJackKnife`**.
- The RE workflow: **constraint classes → coupling → trailer → jackknife**.
- The standout lesson: the **method count located the difficulty** — `SuspensionTrailer` (99) **out-methods** the
  hero car (45), a non-obvious fact only the bytes reveal — the count is a **difficulty map**.
- **Constraints complete the physics part** — single bodies (Ch 41) + wheel forces (Ch 42) + contacts (Ch 43) +
  **linked bodies** (this chapter) — the full rigid-body dynamics.
- The sim can now do everything from a **lone spinning car to an articulated jackknifing rig** — constraints are
  the final extension, from single bodies to *systems*.

**Next:** [Chapter 63 — The Collision World & Spatial Partitioning](../C63-Collision-World/C63-Collision-World.md):
how bodies find what they touch.

**Sources:** `speed.exe` (verified: `Constraint`, `Joint`, `JointDetached`; `RBTrailer`
[C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md); `SuspensionTrailer` `0x008ABCE0`/99 methods; `Trailer`/
`Trailers`/`TrailerPos`/`TrailerParkMoment`; `JackKnife`, `AIActionJackKnife`
[C46.5](../C46-AI-Goals-Actions/05-action-menu.md)).
