# C39.1 — The Per-Frame Pipeline

> **The one-sentence version:** a vehicle simulates through a fixed call chain each frame — sim driver
> (`0x6BB8F0`) → step (`0x6BB5C0`) → `Physics::Simulate` (`0x6BB4D0`) → `IntegrateMotion` (`0x6BA510`) — reached
> from `FrameTick`.

[← Chapter 39 hub](C39-Vehicle-Simulation.md) · [Next: C39.2 — Physics::Simulate →](02-simulate.md)

---

## The call chain

Every frame, `FrameTick` ([Chapter 37](../C37-Frame-Spine-Modules/04-frametick.md)) drives the vehicle sim
through a fixed chain of functions, all verified in `speed.exe`:

```
0x6BB8F0  sim driver        — iterate the bodies that need simulating
   → 0x6BB5C0  step          — per-body step wrapper
      → 0x6BB4D0  Physics::Simulate    — the per-body tick (C39.2)
         → 0x6BA510  Physics::IntegrateMotion   — integrate + move (C39.4)
```

The **sim driver** (`0x6BB8F0`) is one of `FrameTick`'s ~40 calls ([C37.5](../C37-Frame-Spine-Modules/05-module-order.md)):
it walks the set of physics bodies and steps each. The **step** (`0x6BB5C0`) wraps a single body's update. The
**`Physics::Simulate`** (`0x6BB4D0`) is the actual tick ([C39.2](02-simulate.md)), which culminates in
**`IntegrateMotion`** (`0x6BA510`) moving the body ([C39.4](04-integrate.md)).

## Driver → step → simulate → integrate

The four levels are a clean separation of concerns:

- **Driver** (`0x6BB8F0`) — the *population* level: which bodies simulate this frame. It iterates the active
  physics bodies (cars, props) and calls step for each.
- **Step** (`0x6BB5C0`) — the *per-body* wrapper: sets up one body's simulation and calls its `Simulate`.
- **Simulate** (`0x6BB4D0`) — the *body tick* ([C39.2](02-simulate.md)): gate, pre-sim parts, update contacts,
  integrate.
- **IntegrateMotion** (`0x6BA510`) — the *integrate* step ([C39.4](04-integrate.md)): forces → motion.

So the pipeline zooms from "all the bodies" down to "this body's new position," one level at a time. This is the
frame's vehicle work, and it runs for every simulated car (the player, cops, traffic).

> ✅ *Verified:* the chain `0x6BB8F0 → 0x6BB5C0 → 0x6BB4D0 (Physics::Simulate) → 0x6BA510 (IntegrateMotion)` is
> recovered from the disassembly; `Physics::Simulate` and `IntegrateMotion` are confirmed by their bytes
> ([C39.2](02-simulate.md), [C39.4](04-integrate.md)).

## Anchored by structure, not strings

A key RE fact: the vehicle sim is anchored by **structure, not strings**. The names `RigidBody`, `Simulate`,
`CollisionWorld` exist in `.rdata` but aren't code-referenced at the sim sites — the pipeline is identified by
following the call chain and the class structure ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)),
not by string references. So you find `Physics::Simulate` by tracing `FrameTick`'s sim driver down, and confirm
it by its behaviour (gates, part iteration, integrate call), not by a string. This is typical of a shipped
release build where debug strings are present but the code uses direct calls.

## Where it fits in the frame

The sim driver is one of `FrameTick`'s calls ([C37.5](../C37-Frame-Spine-Modules/05-module-order.md)), and its
place in the order matters:

- **After input and AI** ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) — the sim uses this
  frame's driver decisions (throttle, steer).
- **Before rendering** — the renderer draws the car at its new position.
- **With collision** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) — the contact update
  (`0x6A7110`, [C39.2](02-simulate.md)) ties the sim to collision.

So the vehicle sim plugs into the frame after the systems that decide what the car does, before the systems that
show it — exactly where the pipeline needs its inputs ready and its outputs consumed
([C37.5](../C37-Frame-Spine-Modules/05-module-order.md)).

## RE implications

- **The pipeline is `0x6BB8F0 → 0x6BB5C0 → 0x6BB4D0 → 0x6BA510`** — driver → step → simulate → integrate.
- **Find it from `FrameTick`'s sim driver call** ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)),
  not from strings — the sim is structure-anchored.
- **Each level narrows scope** — population → body → tick → integrate.
- **It plugs into the frame** after input/AI, before rendering ([C37.5](../C37-Frame-Spine-Modules/05-module-order.md)).

---

### Key takeaways

- A vehicle simulates through `sim driver (0x6BB8F0) → step (0x6BB5C0) → Physics::Simulate (0x6BB4D0) →
  IntegrateMotion (0x6BA510)`.
- The four levels narrow from the body population, to one body, to its tick, to its integrate step.
- The sim is anchored by **structure, not strings** — find it by the call chain, confirm by behaviour.
- The sim driver is one of `FrameTick`'s calls, placed after input/AI and before rendering.
- Every simulated car (player, cop, traffic) runs this pipeline each frame.

**Continue:** [C39.2 — Physics::Simulate](02-simulate.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md)
