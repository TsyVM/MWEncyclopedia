# Chapter 62 — Physics Constraints, Joints & Trailers

> **Goal of this chapter:** decode the linked-body physics — the `Constraint`/`Joint` system that connects rigid
> bodies, the `RBTrailer` articulated towed body (with the hardest suspension in the game), and the `JackKnife`
> that turns a truck-and-trailer into a pursuit roadblock.

Most bodies simulate alone ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)); but some are
*linked* — a trailer to its truck, coupled by a hitch. This chapter decodes the constraint/joint system that
connects rigid bodies, the `RBTrailer` articulated body it enables, and the dramatic use of a jackknifing truck as
a roadblock. It's the physics of bodies that move *together*, the hardest case in the sim — a multi-axle towed box
on a swinging hitch.

> **Verified against the executable.** The linked-body physics is named in `speed.exe`: **`Constraint`** (the
> constraint), **`Joint`**/**`JointDetached`** (joints and their detachment), **`RBTrailer`**
> ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md), the trailer body), `Trailer`/`Trailers`/`TrailerPos`/
> `TrailerParkMoment`, and **`JackKnife`** (tied to `AIActionJackKnife`,
> [C46.5](../C46-AI-Goals-Actions/05-action-menu.md)). The trailer's suspension is **`SuspensionTrailer`** — 99
> methods, the *most* of the suspension family ([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)),
> because a swinging towed box is the hardest body to stabilise.

---

## Deep-dive pages

- [C62.1 — The constraint system](01-constraints.md): `Constraint`/`Joint` linking bodies.
- [C62.2 — Joints & coupling](02-joints-coupling.md): the hitch, and `JointDetached`.
- [C62.3 — Trailers](03-trailers.md): `RBTrailer` and the hardest suspension.
- [C62.4 — The jackknife](04-jackknife.md): the truck-and-trailer roadblock.
- [C62.5 — Reading constraints in RE](05-reading-constraints.md): navigating the linked-body physics.

---

## 62.1 The constraint system

A **`Constraint`** ([C62.1](01-constraints.md)) is a rule that *links* two rigid bodies
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) — restricting how they move relative to each
other. A **`Joint`** is a constraint that connects bodies at a point (a hinge, a ball joint). The physics solver
([C41.5](../C41-Physics-RigidBody/05-integrate-math.md)) enforces the constraints each frame, so linked bodies move
together correctly — the trailer follows the truck, the hitch holds.

## 62.2 Joints & coupling

A **`Joint`** ([C62.2](02-joints-coupling.md)) couples a trailer to its truck at the *hitch* — a point where the
trailer can pivot but not separate. **`JointDetached`** (verified) is the *broken* state — the hitch coming apart
(a trailer breaking loose on a hard impact, [Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)). So
the coupling is a joint that can hold (normal) or detach (broken) — the trailer is linked until the link fails.

## 62.3 Trailers

**`RBTrailer`** ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) is the articulated towed body — a trailer
pulled behind a truck, coupled by a joint ([C62.2](02-joints-coupling.md)). It's the *hardest* body to simulate:
its suspension, **`SuspensionTrailer`**, has **99 methods** — the most of the family
([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) — because a multi-axle box on a swinging hitch
is a genuinely difficult stabilisation problem ([C62.3](03-trailers.md)). Trailers are the sim's showcase of
articulated physics.

## 62.4 The jackknife

The dramatic use of a truck-and-trailer is the **`JackKnife`** ([C62.4](04-jackknife.md)) — a semi jackknifing
across the road as a *roadblock* ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).
The verified `AIActionJackKnife` ([C46.5](../C46-AI-Goals-Actions/05-action-menu.md)) is the AI action that
executes it — a cop-directed truck swinging its trailer to block your path. So the articulated physics
([C62.3](03-trailers.md)) is not just for ambient trucks — it's a *pursuit weapon*, the jackknife blocking the
road with a linked-body maneuver.

---

### Key takeaways

- A **`Constraint`** links two rigid bodies; a **`Joint`** connects them at a point (hitch/hinge) — the solver
  enforces them so linked bodies move together.
- The truck-trailer **coupling** is a `Joint` — holding normally, detaching (`JointDetached`) on a hard impact.
- **`RBTrailer`** is the articulated towed body — the **hardest** to simulate, with the **most-methoded suspension**
  (`SuspensionTrailer`, 99) in the game.
- The **`JackKnife`** (`AIActionJackKnife`) turns a truck-and-trailer into a **pursuit roadblock** — a cop-directed
  jackknife blocking your path.
- Linked-body physics is the sim's **articulated showcase** — and a dramatic pursuit element.

**Next:** [Chapter 63 — The Collision World & Spatial Partitioning](../C63-Collision-World/C63-Collision-World.md):
how bodies find what they touch.
