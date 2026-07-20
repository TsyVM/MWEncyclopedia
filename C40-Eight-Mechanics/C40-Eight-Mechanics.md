# Chapter 40 — The Eight Vehicle Mechanics

> **Goal of this chapter:** decode the eight `BEHAVIOR_MECHANIC_*` components a Most Wanted car is built from —
> RIGIDBODY, AI, INPUT, ENGINE, SUSPENSION, DAMAGE, DRAW, SOUND — the swappable, data-driven pieces that, run in
> order by `Physics::Simulate`, make a car drive.

A Most Wanted car is not one monolithic object — it's a **composition of eight mechanics**, each a behaviour
component with a defined job. The physics tick ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
runs them in order; the vault ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) tunes each; the
class system ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) constructs them. This chapter
names the eight, explains what each does, and shows how their composition is what lets the game ship dozens of
distinct cars and a shared physics core.

> **Verified against the game data.** All eight mechanics are present in `GLOBAL/attributes.bin` as a
> **contiguous, null-separated string table** — in file order: `BEHAVIOR_MECHANIC_RIGIDBODY` (offset 17110),
> `_AI` (17138), `_INPUT` (17159), `_ENGINE` (17183), `_SUSPENSION` (17208), `_DAMAGE` (17237), `_DRAW` (17262),
> `_SOUND` (17285) — each exactly once. The table sits between the part name `k500_hood` and `DamageVehicle`,
> i.e. inside the vehicle-behaviour schema. The mechanics run on the sim pipeline from
> [Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md) (`Physics::Simulate 0x6BB4D0 → IntegrateMotion
> 0x6BA510`).

---

## The eight mechanics at a glance

| # | Mechanic | Job | Stage |
|---|---|---|---|
| 1 | `BEHAVIOR_MECHANIC_RIGIDBODY` | the rigid-body physics — mass, velocity, integrate | the body ([C40.2](02-rigidbody.md)) |
| 2 | `BEHAVIOR_MECHANIC_AI` | the AI driver — decides controls for cops/traffic | driver ([C40.3](03-ai-and-input.md)) |
| 3 | `BEHAVIOR_MECHANIC_INPUT` | the player input — pad/wheel → controls | driver ([C40.3](03-ai-and-input.md)) |
| 4 | `BEHAVIOR_MECHANIC_ENGINE` | the engine + drivetrain — controls → wheel torque | power ([C40.4](04-engine.md)) |
| 5 | `BEHAVIOR_MECHANIC_SUSPENSION` | the suspension — wheel loads, weight transfer | grip ([C40.5](05-suspension.md)) |
| 6 | `BEHAVIOR_MECHANIC_DAMAGE` | damage — collision → deformation, performance loss | consequence ([C40.6](06-damage-draw-sound.md)) |
| 7 | `BEHAVIOR_MECHANIC_DRAW` | draw — the car's visual representation | presentation ([C40.6](06-damage-draw-sound.md)) |
| 8 | `BEHAVIOR_MECHANIC_SOUND` | sound — engine/tyre/collision audio | presentation ([C40.6](06-damage-draw-sound.md)) |

---

## Deep-dive pages

- [C40.1 — The mechanic model](01-the-mechanic-model.md): what a mechanic is, the verified string table, how
  composition works.
- [C40.2 — RIGIDBODY: the physical body](02-rigidbody.md): the rigid-body mechanic and the integrate.
- [C40.3 — AI & INPUT: the two drivers](03-ai-and-input.md): the swappable driver mechanics.
- [C40.4 — ENGINE: power & drivetrain](04-engine.md): controls to wheel torque.
- [C40.5 — SUSPENSION: loads & weight transfer](05-suspension.md): the grip mechanic.
- [C40.6 — DAMAGE, DRAW & SOUND: consequence & presentation](06-damage-draw-sound.md): the three readers of sim
  state.
- [C40.7 — Reading the mechanics in RE](07-reading-mechanics.md): finding and navigating the mechanic set.

---

## 40.1 What a mechanic is

A **mechanic** is a behaviour component: a piece of a car's behaviour with a name (`BEHAVIOR_MECHANIC_*`), a set
of tuning parameters in the vault ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)), and a per-frame
update. A car *has* eight of them, and its behaviour is the sum of their updates. That the eight names sit as a
contiguous string table in `attributes.bin` ([C40.1](01-the-mechanic-model.md)) reflects that they're an
*enumerated set* — the schema's list of the behaviours a vehicle is composed of.

## 40.2 RIGIDBODY

`BEHAVIOR_MECHANIC_RIGIDBODY` is the **physical body** — the mechanic that owns the car's mass, velocity, and
transform, and runs the integrate ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md),
[Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)). It's the mechanic the other seven act *on*: the
engine pushes it, the suspension holds it up, damage dents it, draw renders it, sound follows its motion. The
rigid body is the car's physical presence ([C40.2](02-rigidbody.md)).

## 40.3 AI & INPUT

Two mechanics produce the **driver's controls** — throttle, brake, steer — and a car has one or the other active:
`BEHAVIOR_MECHANIC_INPUT` reads the **player's** pad/wheel, `BEHAVIOR_MECHANIC_AI` runs the **AI driver**
([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) for cops and traffic. That they're *both*
present as mechanics is the design's elegance: a car is driven by whichever mechanic is active, and the rest of
the car (engine, suspension, body) is identical regardless ([C40.3](03-ai-and-input.md)).

## 40.4 ENGINE

`BEHAVIOR_MECHANIC_ENGINE` is the **powertrain** — it takes the driver's throttle and, via the engine's RPM/power
curve and the drivetrain's gearing ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)),
produces the torque delivered to the wheels. It's the mechanic that makes a car *fast* — its parameters (power,
gearing, redline) are what distinguish a tuner from a supercar ([C40.4](04-engine.md)).

## 40.5 SUSPENSION

`BEHAVIOR_MECHANIC_SUSPENSION` is the **grip** mechanic — it models the springs/dampers at each wheel, computing
the load on each tyre and the weight transfer under acceleration, braking, and cornering
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)). It's what gives a car its
*handling character* — stiff and flat, or soft and rolling ([C40.5](05-suspension.md)).

## 40.6 DAMAGE, DRAW & SOUND

Three mechanics **read** the sim's state and produce consequences/presentation:
`BEHAVIOR_MECHANIC_DAMAGE` turns collisions into deformation and performance loss
([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)); `BEHAVIOR_MECHANIC_DRAW` renders the car;
`BEHAVIOR_MECHANIC_SOUND` produces its audio ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).
These are the *output* mechanics — they don't drive the physics, they express it ([C40.6](06-damage-draw-sound.md)).

## 40.7 Composition

The power of the eight-mechanic model is **composition** ([C40.7](07-reading-mechanics.md)): a car is assembled
from eight components, each tuned by data ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) and
swappable (player vs. AI driver). This is why Most Wanted can have a garage full of distinct cars on one physics
engine — each car is a different *set of parameters* for the same eight mechanics, and a cop is the same car with
the AI mechanic instead of input.

---

### Key takeaways

- A Most Wanted car is a **composition of eight `BEHAVIOR_MECHANIC_*`** components — verified as a contiguous
  string table in `attributes.bin` (RIGIDBODY, AI, INPUT, ENGINE, SUSPENSION, DAMAGE, DRAW, SOUND).
- **RIGIDBODY** is the physical body the others act on; **AI/INPUT** are the swappable drivers; **ENGINE** is
  power; **SUSPENSION** is grip.
- **DAMAGE, DRAW, SOUND** are the output mechanics — they read the sim and produce consequence/presentation.
- `Physics::Simulate` ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) runs the mechanics in
  order; the vault tunes each.
- **Composition** is the design win — many distinct cars, and cop-vs-player, from one physics core by swapping
  parameters and the driver mechanic.

**Next:** [Chapter 41 — Physics & Rigid-Body Dynamics](../C41-Physics-RigidBody/C41-Physics-RigidBody.md): the
rigid-body substrate the mechanics run on.
