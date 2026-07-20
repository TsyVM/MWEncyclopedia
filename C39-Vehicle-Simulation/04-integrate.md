# C39.4 — IntegrateMotion

> **The one-sentence version:** `IntegrateMotion` (`0x6BA510`) is the integrator — a large math frame
> (`sub esp, 0x530`) that turns the accumulated forces into a change in velocity and position, computes
> **speed = |velocity|** via `Math::Sqrt` (`0x5C5E80`), and writes the new **transform at `[this+0xF0]`**.

[← C39.3 — The part & wheel array](03-part-array.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md) ·
[Next: C39.5 — The one-way connector boundary →](05-connectors.md)

---

## The integrator

After the parts have accumulated their forces ([C39.3](03-part-array.md)) and the contacts are updated
([C39.2](02-simulate.md)), the body's motion is advanced by **`IntegrateMotion` at `0x6BA510`**. This is the step
that makes the car actually *move*: it takes the current velocity and the accumulated force/torque and integrates
them over the timestep `dt` to produce a new velocity and a new position/orientation.

Its prologue is verified:

```asm
0x6BA510  Physics::IntegrateMotion:
    81 EC 30 05 00 00    sub  esp, 0x530     ; a large stack frame — the integrator's math scratch
    ...
```

The **`sub esp, 0x530`** (1328 bytes of stack) is the signature of a heavy math routine: it needs room for the
temporary vectors and matrices of rigid-body integration ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md))
— the velocity update, the position update, the orientation update, and the intermediate cross products and
matrix multiplies.

> ✅ *Verified:* `IntegrateMotion` at `0x6BA510` opens `81 EC 30 05 00 00` (`sub esp, 0x530`) — the large math
> frame of the integrator. It calls `Math::Sqrt (0x5C5E80)` to compute speed from velocity, and writes the body
> transform at `[this+0xF0]`.

## What integration does

The integration is the classic rigid-body update ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)):

**1. Velocity update.** `velocity += (force / mass) · dt` — the accumulated force (from the wheels, gravity,
drag) divided by mass, times the timestep, added to velocity. Similarly angular velocity from torque and inertia.

**2. Speed.** `speed = |velocity|` — the magnitude of the velocity vector, via `Math::Sqrt (0x5C5E80)`. This
scalar speed is what the HUD ([Chapter 25](../C27-FrontEnd-Shell-UI/04-hud.md)) shows, what the AI reads, and
what many systems key off ([C39.5](05-connectors.md)).

**3. Position update.** `position += velocity · dt` — the new position from the new velocity. And the orientation
from angular velocity.

**4. Write the transform.** The new position and orientation are written to the body's **transform at
`[this+0xF0]`** ([C39.3](03-part-array.md)) — the car has moved.

So `IntegrateMotion` is where forces become motion: it consumes the frame's accumulated forces and produces the
car's new place in the world.

## `dt` — the timestep

The integration is over a timestep `dt` (the frame's delta time,
[C37.4](../C37-Frame-Spine-Modules/04-frametick.md)). The frame's `dt` is the global at `[0x9259BC]`
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) — read widely across the frame, including
by the sim. A larger `dt` (a slower frame) advances the car further per frame; the physics uses `dt` to stay
time-correct regardless of frame rate ([C37.4](../C37-Frame-Spine-Modules/04-frametick.md)). This is why the sim
reads the frame delta — the integrate step needs it to scale the motion.

> 🟡 *Reasoned:* the specific integration scheme (the exact velocity/position update, fixed vs. variable
> sub-steps) is the standard semi-implicit rigid-body integration, consistent with the verified large math frame,
> the `Math::Sqrt` speed computation, and the `dt` global; the exact per-line math is deeper RE.

## Speed: the output everyone reads

The **speed** computed here (`|velocity|`, via `Math::Sqrt`) is one of the sim's most-consumed outputs:

- **The HUD speedometer** ([Chapter 25](../C27-FrontEnd-Shell-UI/04-hud.md)) displays it.
- **The AI** ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) reads it to decide (matching
  speed, judging gaps).
- **The pursuit system** ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) uses it (escaping requires
  distance, which requires speed).
- **The audio** ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) — engine pitch tracks
  speed/RPM.

So the scalar speed that `IntegrateMotion` computes flows out through the connectors ([C39.5](05-connectors.md))
to the whole game. The physics produces the car's motion; the rest of the game reads that motion — and speed is
the headline number.

## RE implications

- **`IntegrateMotion (0x6BA510)`** is the integrator — `sub esp, 0x530` (large math frame).
- **It integrates forces → velocity → position** over `dt` ([0x9259BC], [C37.4](../C37-Frame-Spine-Modules/04-frametick.md)),
  and writes the **transform at `[this+0xF0]`**.
- **It computes speed = |velocity|** via `Math::Sqrt (0x5C5E80)` — the output the HUD/AI/pursuit/audio read.
- **This is where forces become motion** — the last step of the frame's simulation.

---

### Key takeaways

- `IntegrateMotion (0x6BA510)` is the **integrator** — its `sub esp, 0x530` frame holds the rigid-body math.
- It turns **accumulated forces into velocity into position** over the frame's `dt`, writing the **transform at
  `[this+0xF0]`**.
- It computes **speed = |velocity|** via `Math::Sqrt (0x5C5E80)` — the scalar the whole game reads.
- The timestep `dt` (global `[0x9259BC]`) keeps the motion time-correct regardless of frame rate.
- This is the step where **forces become motion** — the car's new place in the world.

**Continue:** [C39.5 — The one-way connector boundary](05-connectors.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md)
