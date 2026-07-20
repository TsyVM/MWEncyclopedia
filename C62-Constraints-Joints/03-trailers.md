# C62.3 — Trailers

> **The one-sentence version:** `RBTrailer` is the articulated towed body — a trailer with its own mass and
> suspension, coupled to a truck by a hitch joint — and it's the hardest body to simulate, with
> `SuspensionTrailer` (99 methods, the most of the family) stabilising a swinging multi-axle box.

[← C62.2 — Joints & coupling](02-joints-coupling.md) · [Chapter 62 hub](C62-Constraints-Joints.md) ·
[Next: C62.4 — The jackknife →](04-jackknife.md)

---

## RBTrailer: the articulated body

**`RBTrailer`** ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) is the trailer rigid body — a member of the
`RigidBody` tree ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) specialised for a *towed* body:

- **Its own body** — a full rigid body ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) with mass,
  inertia, and a transform, integrated each frame ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)).
- **Its own suspension** — `SuspensionTrailer` ([below](#suspensiontrailer-the-hardest-suspension)) — the trailer's
  wheels/axles.
- **Coupled at the hitch** — a `Joint` ([C62.2](02-joints-coupling.md)) links it to the towing truck.

So a truck-and-trailer is *two* `RigidBody` objects ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) — the
truck (an `RBVehicle`) and the trailer (`RBTrailer`) — joined by a hitch constraint
([C62.1](01-constraints.md)). The trailer isn't part of the truck; it's a *separate body* that follows via the
joint. This is the articulated physics ([C62.1](01-constraints.md)) made concrete: a distinct towed body with its
own dynamics.

> ✅ *Verified:* `RBTrailer` ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) is a rigid-body class;
> `SuspensionTrailer` (vtable `0x008ABCE0`, **99 methods**) is its suspension — the most-methoded of the family
> ([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)). `Trailer`/`Trailers`/`TrailerPos` are
> present.

## SuspensionTrailer: the hardest suspension

The trailer's suspension, **`SuspensionTrailer`**, has **99 methods** — the *most* of the entire suspension family
([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) — more than the hero-car `SuspensionRacer` (45).
This is a verified, telling fact: the *trailer* has the most complex suspension code, because a swinging towed box
is the *hardest body to keep stable*:

- **Multi-axle** — a trailer has multiple axles ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)),
  each with wheels to load and stabilise — more to solve than a car's four wheels.
- **The swinging hitch** — the trailer pivots about the hitch ([C62.2](02-joints-coupling.md)), so its motion is
  *coupled* to the truck's *and* free to swing — a hard dynamic to keep from oscillating or flipping.
- **Stability is difficult** — a trailer wants to *sway* and *jackknife* ([C62.4](04-jackknife.md)); keeping it
  tracking the truck without wild oscillation takes careful suspension/damping code — hence 99 methods.

So the 99-method `SuspensionTrailer` is the sim's admission that *trailers are hard*. That the trailer's suspension
out-methods the hero car's ([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) is a striking
verification result: the method count ([C50.4](../C50-Verification-Methodology/04-vtable-verification.md)) reveals
where the *difficulty* is, and it's the trailer — a multi-axle swinging box is a genuinely harder stabilisation
problem than a sports car. The physics spends its most suspension code on the hardest case.

## Trailers in the world

Trailers appear in Rockport as *world objects* and *pursuit elements*:

- **Ambient trucks** — semis hauling trailers through the city ([Chapter 61](../C61-Traffic-Ambient/C61-Traffic-Ambient.md))
  — big, slow, articulated obstacles you weave around (or crash into,
  [Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)).
- **Scripted moments** — `TrailerParkMoment` (verified) suggests scripted trailer set-pieces (a truck parking, a
  trailer positioned for a scene, [Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)).
- **The jackknife** ([C62.4](04-jackknife.md)) — a truck jackknifing its trailer across the road as a roadblock
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).

So trailers are both *ambient* (trucks in traffic) and *tactical* (the jackknife roadblock). Their articulated
physics ([above](#rbtrailer-the-articulated-body)) makes them *dramatic* — a jackknifing semi or a detaching
trailer ([C62.2](02-joints-coupling.md)) is a spectacular obstacle. That the engine invests the most suspension
code ([above](#suspensiontrailer-the-hardest-suspension)) in getting them right reflects their role as memorable,
physical set-pieces in the world and the chase.

## RE implications

- **`RBTrailer`** is the articulated towed body — a separate rigid body, coupled to the truck by a hitch joint.
- **`SuspensionTrailer` (99 methods)** is the *most-methoded* suspension — a swinging multi-axle box is the hardest
  to stabilise.
- **The method count reveals the difficulty** — the trailer out-methods the hero car, marking it the hard case.
- **Trailers are ambient and tactical** — traffic trucks and the jackknife roadblock ([C62.4](04-jackknife.md)).

---

### Key takeaways

- **`RBTrailer`** is the **articulated towed body** — a separate `RigidBody` with its own mass and suspension,
  coupled to the truck by a hitch **`Joint`** — a truck-and-trailer is *two* bodies.
- Its suspension, **`SuspensionTrailer`**, has **99 methods** — the **most** of the family, more than the hero-car
  `SuspensionRacer` (45) — because a **swinging multi-axle box is the hardest body to stabilise**.
- The **method count reveals the difficulty** ([C50.4](../C50-Verification-Methodology/04-vtable-verification.md)) —
  the trailer, not the sports car, has the most suspension code.
- Trailers are both **ambient** (semis in traffic) and **tactical** (the jackknife roadblock,
  [C62.4](04-jackknife.md)) — and **dramatic** (jackknifing, detaching).
- The engine spends its **most suspension code** on trailers — reflecting their role as memorable physical
  set-pieces.

**Continue:** [C62.4 — The jackknife](04-jackknife.md) · [Chapter 62 hub](C62-Constraints-Joints.md)
