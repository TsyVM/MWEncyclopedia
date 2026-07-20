# Chapter 39 — Vehicle Simulation, End to End

> **Goal of this chapter:** follow a car through one frame of simulation — from the sim driver that calls
> `Physics::Simulate`, through the per-part pre-sim and contact update, to `IntegrateMotion` that moves the
> body — and understand the one-way connector boundary that wires it to the rest of the game.

The vehicle is the heart of Most Wanted, and it is a **physics object**: a `RigidBody`-style body built on a
multi-interface `Physics_Base`, ticked every frame through `Physics::Simulate → IntegrateMotion`. This chapter
is the *end-to-end pipeline* — the frame's worth of work that turns input and forces into a moved car. The
pieces (the eight mechanics, the rigid body, suspension/tyres) get their own chapters
([40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)–[42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md));
here is how they run together in a frame.

> **Verified against the executable.** The sim pipeline is `sim driver (0x6BB8F0) → step (0x6BB5C0) →
> Physics::Simulate (0x6BB4D0) → IntegrateMotion (0x6BA510)`. Confirmed live: `Physics::Simulate` at `0x6BB4D0`
> is `__thiscall` (`push esi; mov esi,ecx; mov eax,[esi]; call [eax+…]` — a virtual dispatch on the body);
> `IntegrateMotion` at `0x6BA510` opens `sub esp, 0x530` (a large math frame — the integrator);
> `Physics_Base::ctor (0x6B9920)` embeds three interface sub-objects. The car is anchored by *structure*, not
> strings. ImageBase `0x400000`, RVA == file-offset.

---

## Deep-dive pages

- [C39.1 — The per-frame pipeline](01-pipeline.md): the sim driver chain that reaches `Physics::Simulate`.
- [C39.2 — Physics::Simulate](02-simulate.md): the per-body tick, its gates, and what it calls.
- [C39.3 — The part & wheel array](03-part-array.md): the `[this+0xEC]` parts iterated in the pre-sim.
- [C39.4 — IntegrateMotion](04-integrate.md): computing speed and moving the transform.
- [C39.5 — The one-way connector boundary](05-connectors.md): how the sim is wired to the game, one direction.
- [C39.6 — Input to tyres, end to end](06-input-to-tyres.md): the whole flow through the eight mechanics.
- [C39.7 — Reading the vehicle sim in RE](07-reading-sim.md): navigating the sim by structure.

---

## 39.1 The pipeline

A vehicle simulates through a fixed call chain each frame, reached from `FrameTick`
([Chapter 37](../C37-Frame-Spine-Modules/04-frametick.md)):

```
0x6BB8F0  sim driver          — iterates the bodies to simulate
  → 0x6BB5C0  step             — per-body step wrapper
    → 0x6BB4D0  Physics::Simulate   — the per-body tick
        ├─ vtbl[+0x4C]/[+0x50]  "should simulate?" gates
        ├─ 0x6A7290  part / wheel pre-sim   (iterates the [this+0xEC] part array)
        ├─ 0x6A7110  contact / collision list update
        └─ 0x6BA510  Physics::IntegrateMotion   — integrate + move the transform [this+0xF0]
```

So one frame of a car is: *should it simulate?* → *pre-simulate its parts* → *update its contacts* →
*integrate its motion*. Verified addresses anchor every step ([C39.1](01-pipeline.md)).

## 39.2 Physics::Simulate

`Physics::Simulate` (`0x6BB4D0`) is the **per-body tick** — `__thiscall`, taking the body in `ECX`, immediately
dispatching a virtual on it ([C39.2](02-simulate.md)). It first checks **gates** (`vtbl[+0x4C]`/`[+0x50]` —
"should this body simulate this frame?"), then runs the **part pre-sim** (`0x6A7290`), the **contact update**
(`0x6A7110`), and finally **`IntegrateMotion`** (`0x6BA510`). So `Simulate` is the orchestrator: gate, pre-sim
parts, update contacts, integrate.

## 39.3 The part & wheel array

A vehicle body is not a point — it's a body with **parts** (wheels, and other simulated components), and the
pre-sim (`0x6A7290`) **iterates the part array at `[this+0xEC]`** ([C39.3](03-part-array.md)). Each part
contributes to the body's simulation — wheels compute suspension and tyre forces
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)), other parts their
dynamics. So the pre-sim is where the car's *components* do their per-frame computation before the body
integrates.

## 39.4 IntegrateMotion

`IntegrateMotion` (`0x6BA510`) is the **integrator** — its large stack frame (`sub esp, 0x530`) holds the math
that advances the body's motion over the timestep ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)):
it computes **speed = |velocity|** (via `Math::Sqrt` `0x5C5E80`), applies the accumulated forces, and writes the
new **transform at `[this+0xF0]`** ([C39.4](04-integrate.md)). So `IntegrateMotion` is where the forces the
parts produced become an actual movement of the car — the final step of the frame's simulation.

## 39.5 The one-way connector boundary

The vehicle sim is wired to the rest of the game through **connectors** ([C39.5](05-connectors.md)) — the
connector class family ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md), 8 classes at `0x00988EC0`).
Crucially, the boundary is **one-way**: data flows *into* the sim (input, AI decisions) and the sim's results
flow *out* (position, speed) through connectors that pass data in a single direction, keeping the physics
isolated from the systems that read it. This isolation is what lets the sim run deterministically without the
consumers reaching in.

## 39.6 Input to tyres

End to end, a frame turns **input into tyre forces into motion** ([C39.6](06-input-to-tyres.md)):

```
input / AI (Ch 46) → the AI/INPUT mechanic → engine mechanic (power) → drivetrain → wheels
   → suspension + tyre forces (Ch 42) → RigidBody integrates (Ch 41) → the car moves
```

The eight mechanics ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) are the stages; `Simulate`
runs them in order; `IntegrateMotion` applies the result. So the pipeline is the mechanics composed into a
frame.

---

### Key takeaways

- A vehicle is a **physics body** ticked each frame: `sim driver (0x6BB8F0) → step → Physics::Simulate
  (0x6BB4D0) → IntegrateMotion (0x6BA510)` — verified.
- `Physics::Simulate` gates ("should simulate?"), pre-sims the parts (`0x6A7290`), updates contacts
  (`0x6A7110`), then integrates.
- The **part array at `[this+0xEC]`** holds the wheels/components each part pre-sims; the **transform at
  `[this+0xF0]`** is what `IntegrateMotion` writes.
- `IntegrateMotion (0x6BA510)` computes speed (`Math::Sqrt 0x5C5E80`) and moves the body — forces become
  motion.
- The sim is wired via **one-way connectors** — data in (input/AI), results out (position/speed), isolating the
  physics.

**Next:** [Chapter 40 — The Eight Vehicle Mechanics](../C40-Eight-Mechanics/C40-Eight-Mechanics.md): the swappable
components a car is built from.
