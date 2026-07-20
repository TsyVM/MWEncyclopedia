# C39.7 — Reading the Vehicle Sim in RE

> **The one-sentence version:** navigate the vehicle sim by **structure, not strings** — start at `FrameTick`'s
> sim driver, follow it down to `Physics::Simulate (0x6BB4D0)` and `IntegrateMotion (0x6BA510)`, and read a car's
> live state from the body's part array `[this+0xEC]` and transform `[this+0xF0]`.

[← C39.6 — Input to tyres, end to end](06-input-to-tyres.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md) ·
[Next: Chapter 40 — The Eight Vehicle Mechanics →](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)

---

## Anchors for sim RE

Reverse-engineering the vehicle sim starts from fixed anchors:

- **`FrameTick`'s sim driver** (`0x6BB8F0`, [C39.1](01-pipeline.md)) — the entry to the pipeline, one of
  `FrameTick`'s calls ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)).
- **`Physics::Simulate (0x6BB4D0)`** ([C39.2](02-simulate.md)) — the per-body tick; its `56 8B F1 8B 06 57 FF 50`
  prologue confirms it.
- **`IntegrateMotion (0x6BA510)`** ([C39.4](04-integrate.md)) — the integrator; its `sub esp, 0x530` confirms it.
- **The body fields** — part array `[this+0xEC]` ([C39.3](03-part-array.md)), transform `[this+0xF0]`
  ([C39.4](04-integrate.md)).
- **`Physics_Base::ctor (0x6B9920)`** — embeds the interface sub-objects
  ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)).

From these, the whole sim is navigable: the entry, the tick, the integrate, and the body's state.

## Structure, not strings

The defining discipline of sim RE is that the pipeline is anchored by **structure, not strings**
([C39.1](01-pipeline.md)). The names (`RigidBody`, `Simulate`) exist in `.rdata` but aren't referenced at the sim
sites, so you can't grep your way to `Physics::Simulate`. Instead:

1. **Start at the frame.** `FrameTick` ([C37.4](../C37-Frame-Spine-Modules/04-frametick.md)) calls the sim driver
   (`0x6BB8F0`) — find it among `FrameTick`'s calls.
2. **Follow the chain down.** Driver → step (`0x6BB5C0`) → `Simulate` (`0x6BB4D0`) → `IntegrateMotion`
   (`0x6BA510`) ([C39.1](01-pipeline.md)).
3. **Confirm by behaviour.** `Simulate` is the `__thiscall` that dispatches a virtual on the body and calls the
   part pre-sim + integrate; `IntegrateMotion` is the big-frame function that calls `Math::Sqrt` and writes
   `[this+0xF0]`. The behaviour, not a string, identifies each.

So you read the sim top-down, from the frame loop into the physics, confirming each function by what it does.

## Reading a car from a memory dump

With the body layout ([C39.3](03-part-array.md)), a car's live state is legible in a dump:

- **Find a body.** The physics bodies are in the class families
  ([C32.6](../C32-Runtime-Class-System/06-reading-binary.md)) — walk the `RigidBody`-family list
  ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) to enumerate the active cars.
- **Read the transform `[this+0xF0]`** — the car's position and orientation *right now*
  ([C39.4](04-integrate.md)). This is where each car is on the map.
- **Walk the part array `[this+0xEC]`** — the wheels/components; read each wheel's state (suspension compression,
  contact, [Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
- **Read the velocity/speed** — the sim's other outputs ([C39.4](04-integrate.md)), the numbers the HUD/AI use.

So a dump's cars are readable: enumerate the bodies, read each one's position (`+0xF0`), parts (`+0xEC`), and
motion. This is how you'd answer "where is the player's car and how fast is it going?" — read the body.

## Verifying the sim

To verify a claim about the sim, you check it against the verified anchors:

- **"Is this `Physics::Simulate`?"** — check the prologue `56 8B F1 8B 06 57 FF 50`
  ([C39.2](02-simulate.md)) and that it calls the part pre-sim + integrate.
- **"Is this the integrator?"** — check `sub esp, 0x530` ([C39.4](04-integrate.md)) and the `Math::Sqrt` call.
- **"Where's the car's position?"** — the transform at `[this+0xF0]` ([C39.3](03-part-array.md)).
- **"What are the mechanics?"** — the `BEHAVIOR_MECHANIC_*` strings in `attributes.bin`
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).

Each claim reduces to a byte pattern, an offset, or a string you can check — the verification-first discipline
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) applied to the sim.

## The sim opens the simulation part

With the pipeline decoded, the rest of the simulation part builds on it: the **eight mechanics**
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) are the stages, the **rigid body**
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) is what integrates, **suspension/tyres**
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) are the wheel forces,
**collision** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) feeds the contact update, and
**AI** ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) provides the inputs. This chapter is the
spine those chapters hang on — the frame's worth of vehicle work, into which each subsystem plugs.

## RE implications

- **Anchor on** the sim driver (`0x6BB8F0`), `Physics::Simulate (0x6BB4D0)`, `IntegrateMotion (0x6BA510)`, and
  the body fields `[this+0xEC]`/`[this+0xF0]`.
- **Read structure, not strings** — top-down from `FrameTick`, confirming each function by behaviour.
- **A car in a dump is legible** — enumerate bodies, read position (`+0xF0`), parts (`+0xEC`), speed.
- **Every claim reduces to a check** — a prologue, an offset, a string.

---

### Key takeaways

- Navigate the sim **top-down from `FrameTick`'s sim driver**, confirming `Physics::Simulate` and
  `IntegrateMotion` by their prologues and behaviour — **structure, not strings**.
- A car's live state is readable from a dump: enumerate bodies, read the **transform `[this+0xF0]`** and **part
  array `[this+0xEC]`** and speed.
- Every sim claim reduces to a **byte pattern, offset, or string** you can verify.
- This chapter is the **spine** of the simulation part — mechanics, rigid body, tyres, collision, and AI all plug
  into this pipeline.

**Next:** [Chapter 40 — The Eight Vehicle Mechanics](../C40-Eight-Mechanics/C40-Eight-Mechanics.md) ·
[Chapter 39 hub](C39-Vehicle-Simulation.md)
