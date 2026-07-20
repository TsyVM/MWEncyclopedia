# Chapter 43 — Collision Detection & Contact Records

> **Goal of this chapter:** decode how a car touches the world — the rigid body's contact update
> (`0x6A7110`, which reads the wheel/part array at `[this+0xEC]`), the contact records it produces, the
> "what did I hit?" classification (`carhitwall`/`carhitcar`/…), and the data-driven reaction records that turn a
> contact into a physical response.

A collision is the moment the sim meets the world. This chapter is the **contact layer**: how the rigid body
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) detects contacts each frame, what a contact
record holds, how contacts are classified by what was hit, and how the `CollisionReactionRecord` /
`AICollisionReactionRecord` vault records turn a contact into push-back, slow-down, and spin. One contact fans out
to three consequences — reaction (physics), damage ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)),
and presentation — all branching on the one classification.

> **Verified against the executable and vault.** The **contact update** is `0x6A7110`, opening `81 EC 28 01 00 00`
> (`sub esp,0x128`) then `56 8B F1 8B 86 EC 00 00 00` — `push esi; mov esi,ecx; mov eax,[esi+0xEC]` — it reads the
> **part/wheel array at `[this+0xEC]`** ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)), tying collision to
> the wheels. The **reaction records are vault records**: in `GLOBAL/attributes.bin`,
> `rh("CollisionReactionRecord")=0x63E3B021` appears **×35**, `rh("AICollisionReactionRecord")=0xAA229CD7` **×14**.
> The **hit classifications are vault keys** — `rh("carhitwall")=0x96BA88FF` ×4, `rh("carhitcar")=0x8FE79512` ×6,
> `rh("carhitsmackable")=0xA906E973` ×1, `rh("carscrapewall")=0x811DE877` ×4. Collision code classes
> `CollisionInstanceList`, `CollisionObjectList`, `WCollisionPack` are present as strings.

---

## Deep-dive pages

- [C43.1 — Detection: the rigid body finds contacts](01-detection.md): the contact update (`0x6A7110`) and the
  collision world.
- [C43.2 — The contact record](02-contact-records.md): point, normal, force, and the two bodies.
- [C43.3 — Classification: what did I hit?](03-classification.md): the `carhit*`/`carscrape*` vault keys.
- [C43.4 — Reaction records](04-reactions.md): `CollisionReactionRecord` (×35) vs. `AICollisionReactionRecord`
  (×14).
- [C43.5 — Smackables](05-smackables.md): `RBSmackable`/`SmackableParams` — the knock-over world objects.
- [C43.6 — Reading collision in RE](06-reading-collision.md): navigating the contact layer.

---

## 43.1 Detection

Collision detection belongs to the **rigid body** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)):
each frame, within `Physics::Simulate` ([C39.2](../C39-Vehicle-Simulation/02-simulate.md)), the **contact update
at `0x6A7110`** tests the body against the world and other bodies. Verified, it reads the **part array at
`[this+0xEC]`** ([C43.1](01-detection.md)) — the wheels and their ground contacts are part of the collision test.
The result is a set of **contacts**: contact point, normal, and impact force.

## 43.2 The contact record

A **contact record** ([C43.2](02-contact-records.md)) describes one contact: the **point** (where), the **normal**
(which way the surface faces), the **force/closing velocity** (how hard), and the **two bodies** involved. The
body holds a list of its current contacts ([C39.2](../C39-Vehicle-Simulation/02-simulate.md)), updated by the
contact update, so the integrate step ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)) can apply the contact
forces. Contacts are the sim's record of "what I'm touching this frame."

## 43.3 Classification

Every contact is **classified by what was hit** ([C43.3](03-classification.md)) — the hinge on which everything
downstream branches. The classification is a vault key: `carhitwall` (a wall), `carhitcar` (another car),
`carhitsmackable` (a knock-over prop, [C43.5](05-smackables.md)), `carscrapewall` (a sustained grind). These are
verified vault keys — their reflection hashes appear in `attributes.bin` (`carhitcar` ×6, `carhitwall` ×4). The
tag decides the reaction, the damage, and the effect/sound.

## 43.4 Reaction records

The **physical response** to a contact is data-driven by two verified vault records
([C43.4](04-reactions.md)): **`CollisionReactionRecord`** (×35 in `attributes.bin`) governs the player/generic
car's response — push-back, slow-down, spin — and **`AICollisionReactionRecord`** (×14) governs the AI car's
response. Splitting player and AI is deliberate: a cop's PIT manoeuvre
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) needs predictable cop collision behaviour, tuned
separately from how a crash feels from the player's seat.

## 43.5 Smackables

The world's **knock-over objects** — cones, signs, fences — are **`RBSmackable`** bodies
([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) with `SmackableParams` tuning
([C43.5](05-smackables.md)). They sit inert until a car hits them (`carhitsmackable`), then react as light rigid
bodies — the satisfying scatter of driving through a construction zone. They're the cheapest physics bodies, and
there are many of them dressing the world ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)).

## 43.6 One contact, three consequences

The chapter's thesis: a single contact **fans out** to three independent reads
([C43.4](04-reactions.md)):

```
contact (tag = carhitwall, force F)
   ├──▶ Reaction   (CollisionReactionRecord)  →  physics: push / slow / spin
   ├──▶ Damage     (Damage* mechanic, Ch 45)  →  zone damage
   └──▶ Presentation (effects + carhit* sound) →  sparks, thud
```

Detection is code (exact, fast); response is data (tunable). The same contact feeds reaction, damage, and
presentation — three reads of one tag — so each can be balanced without disturbing the others.

---

### Key takeaways

- Collision detection is the **rigid body's** job — the **contact update `0x6A7110`** reads the wheel/part array
  at `[this+0xEC]` (verified) and produces contacts.
- A **contact record** holds point, normal, force, and the two bodies; the body keeps a list of its current
  contacts.
- Contacts are **classified by what was hit** — `carhitwall`/`carhitcar`/`carhitsmackable`/`carscrapewall`
  (verified vault keys) — the hinge for everything downstream.
- The response is **data-driven**: `CollisionReactionRecord` (×35, player) and `AICollisionReactionRecord` (×14,
  AI) — tuned separately.
- **Smackables** (`RBSmackable`/`SmackableParams`) are the knock-over props; a contact **fans out** to reaction,
  damage, and presentation.

**Next:** [Chapter 44 — Surfaces: Grip, Sound & Effects](../C44-Surfaces-Grip/C44-Surfaces-Grip.md): driving *on*
the world, the continuous counterpart to hitting it.
