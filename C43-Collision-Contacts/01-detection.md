# C43.1 — Detection: the Rigid Body Finds Contacts

> **The one-sentence version:** the contact update at `0x6A7110` — verified to open `sub esp,0x128` then
> `mov esi,ecx; mov eax,[esi+0xEC]` — tests the body against the world each frame, reading the wheel/part array at
> `[this+0xEC]`, so the wheels' ground contacts are part of collision detection.

[← Chapter 43 hub](C43-Collision-Contacts.md) · [Next: C43.2 — The contact record →](02-contact-records.md)

---

## Collision is the body's job

Collision detection belongs to the **rigid body** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)),
not a separate system holding the body at arm's length. Within the per-body tick
([C39.2](../C39-Vehicle-Simulation/02-simulate.md)), the **contact update at `0x6A7110`** runs — one of the
phases of `Physics::Simulate`, between the part pre-sim and the integrate. Its job is to find everything the body
is touching this frame and record it as contacts.

Its prologue is verified:

```asm
0x6A7110  contact update:
    81 EC 28 01 00 00    sub  esp, 0x128     ; 296-byte frame (collision math scratch)
    56                   push esi
    8B F1                mov  esi, ecx        ; esi = this (__thiscall)
    8B 86 EC 00 00 00    mov  eax, [esi+0xEC] ; eax = this->part_array   ← the wheels!
    8B ...
```

The `mov eax,[esi+0xEC]` is the telling instruction: the contact update **reads the part array at `[this+0xEC]`**
([C39.3](../C39-Vehicle-Simulation/03-part-array.md)) — the same wheel/component array the pre-sim iterates. So
collision detection *includes the wheels*: each wheel's ground contact is found here.

> ✅ *Verified:* the contact update at `0x6A7110` = `81 EC 28 01 00 00 56 8B F1 8B 86 EC 00 00 00` = `sub esp,0x128;
> push esi; mov esi,ecx; mov eax,[esi+0xEC]`. It is a `__thiscall` with a 296-byte frame that reads the part array
> at `[this+0xEC]`.

## Two kinds of contact: wheels and body

That the contact update reads the part array reveals a key structure: a car has **two kinds of contact**:

- **Wheel-ground contacts.** Each wheel ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)) contacts the road —
  this is the *continuous* contact that produces suspension load and tyre grip
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)). The wheels are always in
  contact with *something* (road, grass, kerb), and that contact determines the surface
  ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)).
- **Body-world contacts.** The car's body hitting a wall, another car, or a prop — the *discrete* collisions
  ([C43.3](03-classification.md)) that produce reactions and damage.

The contact update handles both: it reads the wheels (`[this+0xEC]`) to find their ground contacts, and tests the
body hull against the world for impacts. So "detection" spans the gentle (wheels rolling on road) and the violent
(crashing into a wall) — both are contacts the body finds each frame.

## The collision world

The body tests against the **collision world** — the static geometry of the level
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md), [Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md))
plus the other dynamic bodies. The verified code classes `CollisionInstanceList`, `CollisionObjectList`, and
`WCollisionPack` ([C43.6](06-reading-collision.md)) are this system: the world's collision geometry is packed
(`WCollisionPack`) and organised into instance/object lists the body queries. So detection is the body asking the
collision world "what's near me, and am I touching it?" — against both the baked world and the moving cars.

> 🟡 *Reasoned:* the broad-phase/narrow-phase structure of the collision query (spatial partition → hull test) is
> the standard collision-detection design, consistent with the verified `CollisionInstanceList`/`CollisionObjectList`
> classes and the contact-update function; the exact query algorithm is deeper RE. The contact update's `[this+0xEC]`
> read and the collision code classes are verified.

## Why detection is in the body

Putting collision detection in the rigid body (rather than a wholly external physics engine) has a clear logic:

- **The body knows its parts.** The wheels ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)) are the body's,
  and their ground contacts feed suspension/tyres directly — so testing them from the body is natural (it reads
  `[this+0xEC]`).
- **Contacts feed the integrate.** The contacts found here are applied in the *same* body's integrate step
  ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)) — keeping detection and response in one tick keeps the
  physics coherent (no frame lag between "touched" and "reacted").
- **The interface design supports it.** The body implements a collision interface
  ([C41.2](../C41-Physics-RigidBody/02-physics-base.md)), so the collision world holds it as "a collide-able,"
  while the body itself drives the query. The interface is the contract; the body is the implementation.

So detection is a phase of the body's own tick, reading its own parts and testing against the world — the sim and
collision are one integrated pass, not two systems reaching across a boundary.

## RE implications

- **The contact update `0x6A7110`** (`sub esp,0x128`, `__thiscall`) reads the **part array `[this+0xEC]`** —
  wheels are part of detection.
- **Two kinds of contact** — wheel-ground (continuous, → suspension/tyres) and body-world (discrete, → reaction/damage).
- **The collision world** is `WCollisionPack` + `CollisionInstanceList`/`CollisionObjectList` — baked geometry +
  dynamic bodies.
- **Detection lives in the body** — it knows its parts, and contacts feed the same body's integrate.

---

### Key takeaways

- Collision detection is a **phase of the rigid body's tick** — the **contact update `0x6A7110`** (verified `sub
  esp,0x128; mov esi,ecx; mov eax,[esi+0xEC]`).
- It reads the **part array at `[this+0xEC]`** — the **wheels' ground contacts are part of detection**, alongside
  body-world impacts.
- A car has **two kinds of contact**: continuous wheel-ground (→ suspension/tyres) and discrete body-world (→
  reaction/damage).
- The body tests against the **collision world** — `WCollisionPack` + `CollisionInstanceList`/`CollisionObjectList`
  (verified classes).
- Detection lives **in the body** so contacts feed the **same body's integrate** — sim and collision are one
  integrated pass.

**Continue:** [C43.2 — The contact record](02-contact-records.md) · [Chapter 43 hub](C43-Collision-Contacts.md)
