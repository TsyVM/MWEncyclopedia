# C43.2 — The Contact Record

> **The one-sentence version:** a contact record describes one contact — the **point** (where), the **normal**
> (surface facing), the **force / closing velocity** (how hard), and the **two bodies** involved — and the rigid
> body holds a list of its current contacts for the integrate step to resolve.

[← C43.1 — Detection](01-detection.md) · [Chapter 43 hub](C43-Collision-Contacts.md) ·
[Next: C43.3 — Classification →](03-classification.md)

---

## What a contact holds

When the contact update ([C43.1](01-detection.md)) finds the body touching something, it records a **contact** —
a small structure describing that touch. A contact holds, at minimum:

- **The contact point** — where in world space the two surfaces meet.
- **The contact normal** — the direction the contact surface faces (which way the push-apart acts).
- **The penetration / separation** — how far the bodies overlap (or the gap), driving the corrective push.
- **The impact force / closing velocity** — how fast the bodies are coming together, which sets how hard the
  collision is.
- **The two bodies** — the pair in contact (this body, and the world or the other body).

These are the classic quantities of a rigid-body contact ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)):
enough to compute the corrective impulse that keeps the bodies from interpenetrating and imparts the collision's
effect.

> 🟡 *Reasoned:* the contact record's fields (point, normal, penetration, impact, body pair) are the standard
> rigid-body contact structure, consistent with the verified contact update ([C43.1](01-detection.md)) and the
> integrate step that consumes them ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)); the exact byte layout
> and size of the record are deeper RE. The contact *update function* and the body's part-array read are verified.

## The contact list

A body doesn't have one contact — it has **several at once**: four wheels on the ground, plus any body-hull
impacts. So the body holds a **list of current contacts** ([C39.2](../C39-Vehicle-Simulation/02-simulate.md)),
rebuilt each frame by the contact update. The list is the body's snapshot of "everything I'm touching right now":

- **The wheel contacts** — normally four, one per wheel ([C43.1](01-detection.md)) — always present while the car
  is on the ground.
- **Impact contacts** — zero or more, depending on what the body is hitting this frame (a wall, a car, a prop).

The integrate step ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)) walks this list, applying each contact's
force to the body — the wheel contacts hold the car up and provide grip
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)), the impact contacts push it
away from what it hit. So the contact list is the interface between detection (which fills it) and response (which
consumes it).

## Contacts are per-frame

An important property: contacts are **rebuilt each frame**, not persistent objects. The contact update
([C43.1](01-detection.md)) re-tests and re-populates the list every tick, because the world is moving — the car,
the other cars, the wheels all shift, so last frame's contacts may not be this frame's. This is why:

- **A contact is a frame-local fact** — "I'm touching this wall *this frame*," not a lasting relationship.
- **Sustained contact is re-detected** — a scrape along a wall ([C43.3](03-classification.md)) is the *same*
  contact re-found each frame, which is how the game knows the grind is continuing (and keeps the scrape sound and
  effect going).
- **Contacts don't leak** — since the list is rebuilt, a contact that ends simply isn't re-added; there's no
  stale state to clean up.

So the contact list is a fresh, per-frame snapshot — the standard approach for a real-time physics sim, and the
reason collisions feel immediate (they're detected and resolved in the same tick, [C43.1](01-detection.md)).

## From contact to consequence

The contact record is the *input* to the three consequences ([C43.4](04-reactions.md)):

- **Reaction** ([C43.4](04-reactions.md)) reads the contact's force and normal to compute the physical push
  (via `CollisionReactionRecord`).
- **Damage** ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)) reads the contact's force and
  point to decide which zone is hurt and how much.
- **Presentation** reads the contact's point and classification ([C43.3](03-classification.md)) to place the
  spark/thud effect and pick the sound.

So the contact record is the shared data all three consequences branch from — one detection, three reads
([C43.4](04-reactions.md)). Its fields (point, normal, force, classification) are exactly what each consumer
needs: the physics wants force and normal, the damage wants force and point, the presentation wants point and tag.
The record is designed to serve all three.

## RE implications

- **A contact record holds** point, normal, penetration, impact force/closing velocity, and the two bodies — the
  standard contact quantities.
- **The body holds a contact list** — wheels (≈4) plus impacts — rebuilt each frame by the contact update.
- **Contacts are per-frame** — a frame-local fact; sustained contact is the same contact re-found (the scrape).
- **The contact is the input to three consequences** — reaction (force/normal), damage (force/point),
  presentation (point/tag).

---

### Key takeaways

- A **contact record** describes one touch: **point, normal, penetration, impact force/closing velocity**, and the
  **two bodies**.
- The body holds a **contact list** — the ≈4 wheel-ground contacts plus any body-hull impacts — rebuilt every
  frame by the contact update.
- Contacts are **per-frame snapshots**, not persistent — sustained contact (a wall scrape) is the same contact
  re-detected each tick.
- The integrate step ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)) **walks the list**, applying each
  contact's force (wheels hold the car up; impacts push it away).
- The contact record is the **shared input** to the three consequences — reaction, damage, and presentation each
  read the fields they need.

**Continue:** [C43.3 — Classification: what did I hit?](03-classification.md) · [Chapter 43 hub](C43-Collision-Contacts.md)
