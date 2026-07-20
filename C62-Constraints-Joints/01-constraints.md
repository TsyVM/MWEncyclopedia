# C62.1 — The Constraint System

> **The one-sentence version:** a `Constraint` links two rigid bodies by restricting their relative motion, and
> the physics solver enforces it each frame — the mechanism that makes linked bodies (a trailer and its truck)
> move together correctly.

[← Chapter 62 hub](C62-Constraints-Joints.md) · [Next: C62.2 — Joints & coupling →](02-joints-coupling.md)

---

## What a constraint is

A **`Constraint`** is a *rule* that links two rigid bodies ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md))
— a restriction on how they move *relative to each other*. Where a lone body integrates freely
([C39.4](../C39-Vehicle-Simulation/04-integrate.md)), constrained bodies must satisfy their constraint:

- **A hitch constraint** keeps a trailer *attached* to a truck ([C62.3](03-trailers.md)) — they can't drift apart.
- **A hinge constraint** lets bodies rotate about an axis but not separate.
- **The constraint restricts degrees of freedom** — a free body has 6 (3 move + 3 rotate); a constraint removes
  some (a hitch removes the separation freedom).

So a constraint is the physics of *linkage* — the rule that binds bodies together. The verified `Constraint` class
is this mechanism; `Joint` ([C62.2](02-joints-coupling.md)) is a specific constraint (a connection at a point).

> ✅ *Verified:* `Constraint` and `Joint`/`JointDetached` are present in `speed.exe` — the linked-body physics.
> `RBTrailer` ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) is the body they link.

## The solver enforces constraints

Constraints don't enforce themselves — the physics **solver** ([C41.5](../C41-Physics-RigidBody/05-integrate-math.md))
does, each frame:

```
each physics step (Ch 39):
   integrate the bodies freely (C39.4)
   → the constraint is now violated (bodies drifted apart)
   → the solver computes corrective impulses to satisfy the constraint
   → apply them → bodies satisfy the constraint again
```

So the solver *corrects* constraint violations: after the free integrate ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)),
the linked bodies may have drifted (the trailer lagging the truck), and the solver applies impulses to pull them
back into a valid configuration (the hitch reconnected). This is the standard constrained-dynamics approach —
integrate, then solve constraints — and it runs within the physics step
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) for the linked bodies. The result is that a
trailer *follows* its truck smoothly, held by the enforced hitch constraint.

> 🟡 *Reasoned:* the integrate-then-solve constrained-dynamics model is the standard approach, consistent with the
> verified `Constraint`/`Joint` classes and the rigid-body integrator ([C41.5](../C41-Physics-RigidBody/05-integrate-math.md));
> the exact solver (iterative impulse, etc.) is deeper RE. The constraint classes are verified.

## Why constraints matter

Most Wanted uses constraints sparingly (most bodies are free, [Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md))
— but where it does, they enable behaviour impossible with lone bodies:

- **Trailers** ([C62.3](03-trailers.md)) — a truck towing a trailer is *two* bodies constrained at the hitch; the
  articulation (the trailer swinging behind) is the constraint in action.
- **Believable linkage** — a rigidly-attached trailer (one body) couldn't jackknife
  ([C62.4](04-jackknife.md)) or swing; the constraint (two bodies, coupled) gives the *articulated* motion that
  makes a semi look real.
- **Detachment** — because it's a constraint, it can *break* (`JointDetached`, [C62.2](02-joints-coupling.md)) — a
  trailer breaking loose, which a single rigid body couldn't do.

So constraints are the physics of *articulation and linkage* — the reason a truck-and-trailer moves like a
truck-and-trailer (swinging, jackknifing, potentially detaching) rather than a single rigid shape. They're a
specialised but important part of the sim, enabling the articulated bodies
([C62.3](03-trailers.md)) that the pursuit ([C62.4](04-jackknife.md)) and the world use. Constraints are where the
rigid-body physics ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) extends from single bodies to
*systems* of linked bodies.

## RE implications

- **A `Constraint`** links two bodies by restricting relative motion (removing degrees of freedom); a `Joint` is a
  point connection.
- **The solver enforces constraints** — integrate freely, then correct violations with impulses.
- **Constraints enable articulation** — trailers, jackknifing, detachment — impossible with lone bodies.
- **Used sparingly** but essential for the linked-body cases (trailers, [C62.3](03-trailers.md)).

---

### Key takeaways

- A **`Constraint`** links two rigid bodies by **restricting their relative motion** (removing degrees of freedom)
  — the physics of linkage; a **`Joint`** is a constraint connecting bodies at a point (hitch/hinge).
- The **solver enforces constraints each frame** — integrate the bodies freely, then apply corrective impulses to
  satisfy the constraint (the trailer pulled back behind the truck).
- Constraints enable **articulation** — a truck-and-trailer swings, jackknifes, and can detach because it's *two*
  coupled bodies, not one rigid shape.
- They can **break** (`JointDetached`) — a constraint that fails on a hard impact — which a single body couldn't
  do.
- Constraints extend the rigid-body physics from **single bodies to systems of linked bodies** — used sparingly but
  essential for trailers and the jackknife.

**Continue:** [C62.2 — Joints & coupling](02-joints-coupling.md) · [Chapter 62 hub](C62-Constraints-Joints.md)
