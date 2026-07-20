# C63.3 — Narrow-Phase

> **The one-sentence version:** the narrow-phase does the precise shape test on the few candidates the broad-phase
> found — testing `CollisionElement` shapes against each other to produce the exact contact (point, normal,
> penetration) the physics resolves.

[← C63.2 — Broad-phase: AABB & Grid](02-broad-phase.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: C63.4 — CollisionCache & queries →](04-collision-cache.md)

---

## The precise test

The broad-phase ([C63.2](02-broad-phase.md)) gives *candidate pairs* — objects whose AABBs overlap, that *might*
collide. The **narrow-phase** does the *precise* test: the actual geometry check that determines whether they
*really* collide and, if so, *how*. It works on the **`CollisionElement`** shapes ([above](#the-shape-primitives)):

- **Shape vs. shape** — test the two objects' actual collision shapes (`CollisionElement`,
  [C63.1](01-collision-world.md)) for overlap — the precise geometry, not the AABB.
- **Produce the contact** — if they overlap, compute the *contact*
  ([C43.2](../C43-Collision-Contacts/02-contact-records.md)): the contact point, the normal, the penetration
  depth — the data the physics needs to resolve the collision ([C43.4](../C43-Collision-Contacts/04-reactions.md)).

So the narrow-phase is where a "maybe" (from the broad-phase) becomes a "yes, here" (a contact) or a "no"
(the shapes don't actually overlap despite their AABBs). It's the *expensive* stage — precise geometry tests are
costly — which is why the broad-phase ([C63.2](02-broad-phase.md)) works so hard to feed it only a *few*
candidates.

> ✅ *Verified:* `CollisionElement` (the shape primitive) and `CollisionDetection` are present in `speed.exe`; the
> contacts they produce are the records of ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)).

## The shape primitives

Collision shapes are **`CollisionElement`s** — the primitives a body's collision geometry is built from
([C63.1](01-collision-world.md)). Typical collision primitives (the narrow-phase tests pairs of these):

- **Boxes/hulls** — a car's body as a convex hull or box; a building as a box.
- **Planes** — the ground, walls — large flat surfaces.
- **Spheres/capsules** — simple bounds for round objects or wheels.

The narrow-phase has a *test per pair of primitive types* — box-vs-plane (car hitting the ground), hull-vs-hull
(car hitting car, [C43.3](../C43-Collision-Contacts/03-classification.md)), etc. Each pair type has a specific
algorithm (the math for that shape combination). So the narrow-phase is a *dispatch* on the two shapes' types to
the right test, producing the contact. Using *simple* primitives ([above](#the-precise-test)) keeps these tests
fast — a hull-vs-hull test is far cheaper than mesh-vs-mesh, which is why the collision geometry is simplified
([C63.1](01-collision-world.md)). The `CollisionElement` primitives are the vocabulary the narrow-phase tests.

> 🟡 *Reasoned:* the primitive-pair narrow-phase (box/hull/plane/sphere tests) is the standard collision-detection
> model, consistent with the verified `CollisionElement`/`CollisionDetection` and the simplified collision geometry
> ([C63.1](01-collision-world.md)); the exact primitive set and algorithms are deeper RE. The classes are verified.

## Two stages, one purpose

The broad-phase ([C63.2](02-broad-phase.md)) and narrow-phase (this page) together form the *complete* collision
detection ([C43.1](../C43-Collision-Contacts/01-detection.md)):

| Stage | Works on | Cost | Produces |
|---|---|---|---|
| Broad-phase ([C63.2](02-broad-phase.md)) | AABBs in a Grid | cheap | candidate pairs |
| Narrow-phase (this page) | `CollisionElement` shapes | expensive | precise contacts |

The division of labour is *cheap-culls-many, expensive-tests-few*: the broad-phase rules out the vast majority with
cheap AABB tests, and the narrow-phase precisely tests only the handful that survive. This is *the* reason
real-time collision is possible — you can't afford a precise test against everything, but you *can* afford a cheap
cull against everything plus a precise test against a few. The two stages feed the contact records
([C43.2](../C43-Collision-Contacts/02-contact-records.md)) that the physics resolves
([C43.4](../C43-Collision-Contacts/04-reactions.md)) — so this chapter (the spatial *finding*) and Chapter 43 (the
contact *response*) are the two halves of collision: find what touches (63), respond to it (43).

## RE implications

- **The narrow-phase** does the precise shape test on the broad-phase's candidates — producing exact contacts.
- **`CollisionElement`** shapes (boxes/hulls/planes/spheres) are the primitives tested — simple, for speed.
- **The narrow-phase dispatches** on shape-type pairs to the right test algorithm.
- **Broad + narrow** = complete detection — cheap-culls-many, expensive-tests-few — the reason real-time collision
  works.

---

### Key takeaways

- The **narrow-phase** does the **precise geometry test** on the broad-phase's candidate pairs — turning a "maybe"
  into a **contact** (point, normal, penetration, [C43.2](../C43-Collision-Contacts/02-contact-records.md)) or a
  "no."
- It tests **`CollisionElement` shapes** — simple primitives (boxes, hulls, planes, spheres) — dispatching on the
  shape-type pair to the right algorithm.
- Simple primitives keep it **fast** — hull-vs-hull, not mesh-vs-mesh — which is why the collision geometry is
  simplified ([C63.1](01-collision-world.md)).
- **Broad-phase (cheap, many) + narrow-phase (expensive, few)** is the complete detection — the reason real-time
  collision is possible.
- This chapter (spatial **finding**) + [Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md) (contact
  **response**) are the **two halves of collision**.

**Continue:** [C63.4 — CollisionCache & queries](04-collision-cache.md) · [Chapter 63 hub](C63-Collision-World.md)
