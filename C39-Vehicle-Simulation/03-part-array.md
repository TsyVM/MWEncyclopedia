# C39.3 — The Part & Wheel Array

> **The one-sentence version:** a vehicle body owns a **part array at `[this+0xEC]`** — the wheels and simulated
> components — and the pre-sim (`0x6A7290`) iterates it so each part contributes its per-frame forces before the
> body integrates.

[← C39.2 — Physics::Simulate](02-simulate.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md) ·
[Next: C39.4 — IntegrateMotion →](04-integrate.md)

---

## A body is made of parts

A vehicle is not a point mass — it's a body composed of **parts**: four wheels, and other simulated components
(the chassis, and the attachment points the mechanics use,
[Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)). Those parts are held in an array at **`[this+0xEC]`**
on the body, and they're what the pre-sim walks each frame ([C39.2](02-simulate.md)).

The pre-sim (`0x6A7290`) iterates the array: for each part, it runs that part's per-frame computation, which for
a wheel is the suspension compression and the tyre force
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)). Each part accumulates
forces on the body, which the integrate step ([C39.4](04-integrate.md)) then applies.

> ✅ *Verified:* the vehicle body's part array is at `[this+0xEC]` and the body transform at `[this+0xF0]`
> (adjacent fields on the body); the pre-sim iterating the parts is at `0x6A7290`, called from
> `Physics::Simulate` ([C39.2](02-simulate.md)).

## `[this+0xEC]` and `[this+0xF0]`

Two adjacent body fields anchor the sim's data:

| Offset | Field | Role |
|---|---|---|
| `[this+0xEC]` | part array | the wheels/components iterated in the pre-sim |
| `[this+0xF0]` | transform | the body's position/orientation, written by `IntegrateMotion` |

That the part array (`+0xEC`) and the transform (`+0xF0`) are adjacent is the sim's core state laid out
together: *what the body is made of* and *where the body is*. The pre-sim reads/updates the parts (`+0xEC`); the
integrate writes the transform (`+0xF0`). These two fields are the handle to a car's simulation in a memory dump
([C39.7](07-reading-sim.md)).

## Wheels: the important parts

The most important parts are the **wheels** — they're where the car meets the road, and where most of the
interesting physics happens ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)):

- **Suspension.** Each wheel has a suspension that compresses against the ground, producing a normal force that
  holds the car up and controls weight transfer.
- **Tyre forces.** Each wheel produces longitudinal (drive/brake) and lateral (cornering) tyre forces based on
  slip, load, and the surface ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)).
- **Contact.** Each wheel's ground contact ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md))
  determines whether it's on the road, and on what surface.

So iterating the part array is, largely, iterating the wheels — computing four suspensions and four tyre contact
patches, the forces that make a car drive, corner, and brake. The pre-sim is where "the rubber meets the road"
literally happens each frame.

## Parts accumulate, the body integrates

The division of labour between parts and body is clean:

- **Parts accumulate forces.** Each part (wheel) computes its force and adds it to the body's accumulated force
  and torque ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) — at a position (the wheel's
  contact point), so it produces both force and torque on the body.
- **The body integrates once.** After all parts have contributed, `IntegrateMotion` ([C39.4](04-integrate.md))
  integrates the body's total accumulated force/torque into a change in velocity and position — one integrate for
  the whole body.

So the parts are *force producers*, and the body is the *single rigid mass* they act on. Four wheels pushing at
four contact points, integrated as one body, is what gives the car its handling — weight transfer, cornering
balance, and grip all emerge from the wheel forces acting on the shared body
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).

## RE implications

- **The part array is at `[this+0xEC]`** — the wheels/components; the pre-sim (`0x6A7290`) iterates it.
- **The transform is at `[this+0xF0]`** — adjacent to the parts; `IntegrateMotion` writes it.
- **Wheels are the important parts** — suspension + tyre forces + ground contact, computed per wheel.
- **Parts accumulate forces; the body integrates once** — four contact points acting on one rigid mass.

---

### Key takeaways

- A vehicle body owns a **part array at `[this+0xEC]`** — the wheels and simulated components.
- The pre-sim (`0x6A7290`) **iterates the parts**, each contributing its per-frame forces to the body.
- The transform at **`[this+0xF0]`** is adjacent — parts (`+0xEC`) and position (`+0xF0`) are the sim's core
  state together.
- **Wheels are the key parts** — suspension, tyre forces, and ground contact, four per car.
- Parts are **force producers**; the body **integrates once** — the handling emerges from wheel forces on one
  rigid mass.

**Continue:** [C39.4 — IntegrateMotion](04-integrate.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md)
