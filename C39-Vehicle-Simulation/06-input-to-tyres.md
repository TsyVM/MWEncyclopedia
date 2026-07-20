# C39.6 — Input to Tyres, End to End

> **The one-sentence version:** one frame of a car is a chain — input/AI → the AI/INPUT mechanic → engine (power)
> → drivetrain → wheels → suspension + tyre forces → RigidBody integrate → the car moves — the eight mechanics
> composed into the sim pipeline.

[← C39.5 — The one-way connector boundary](05-connectors.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md) ·
[Next: C39.7 — Reading the vehicle sim in RE →](07-reading-sim.md)

---

## The whole chain

Putting the pieces together, a single frame turns a driver's intent into a moved car through a chain of stages:

```
1. input / AI          (Ch 46)  — throttle, brake, steer decided
      ↓  (one-way connector, into the sim — C39.5)
2. AI / INPUT mechanic  (Ch 40)  — read the driver's controls onto the body
      ↓
3. ENGINE mechanic      (Ch 40)  — controls → engine torque (RPM, power curve)
      ↓
4. drivetrain           (Ch 42)  — engine torque → wheel torque (gearing, diff)
      ↓
5. wheels               (Ch 42)  — the part array [this+0xEC] (C39.3)
      ↓
6. suspension + tyres   (Ch 42)  — wheel loads + slip → tyre forces on the body
      ↓
7. RigidBody integrate  (Ch 41)  — IntegrateMotion (0x6BA510): forces → motion (C39.4)
      ↓
8. the car moves                 — new transform [this+0xF0], speed published (C39.5)
```

Each stage is a mechanic or a physics step, and `Physics::Simulate` ([C39.2](02-simulate.md)) is what runs them
in order for one body, one frame.

## The eight mechanics are the stages

The stages 2–6 are the **eight vehicle mechanics** ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md))
— the `BEHAVIOR_MECHANIC_*` components a car is built from:

- **`BEHAVIOR_MECHANIC_AI`** / **`_INPUT`** — stage 2: read the driver's intent (AI for cops/traffic, input for
  the player) onto the body.
- **`BEHAVIOR_MECHANIC_ENGINE`** — stage 3: the engine — controls to torque, via the RPM/power model
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md) for the sound side).
- **`BEHAVIOR_MECHANIC_SUSPENSION`** — stage 6: the suspension — wheel loads, weight transfer.
- **`BEHAVIOR_MECHANIC_RIGIDBODY`** — stage 7: the rigid body — the integrate.
- **`BEHAVIOR_MECHANIC_DAMAGE`** ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)),
  **`_SOUND`** ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)),
  **`_DRAW`** — the presentation/consequence mechanics, reading the sim's state.

So the car is *composed* of mechanics, and a frame *runs* them in order. The pipeline
([C39.1](01-pipeline.md)) is the mechanics executed as a sequence — decide, power, transmit, grip, integrate,
move.

> ✅ *Verified:* all eight `BEHAVIOR_MECHANIC_*` strings — `AI`, `DAMAGE`, `DRAW`, `ENGINE`, `INPUT`, `RIGIDBODY`,
> `SOUND`, `SUSPENSION` — are present in `attributes.bin` as the vehicle's mechanic set
> ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).

## Following one input: throttle

Trace a single input — the throttle — through the chain to see it end to end:

1. **The player presses accelerate** (or the AI decides to,
   [Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)). The throttle value (0..1) is set on the
   body via the input/AI mechanic (stage 2).
2. **The engine mechanic** (stage 3) uses the throttle and current RPM to compute engine torque off the power
   curve ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).
3. **The drivetrain** (stage 4) multiplies engine torque by the current gear ratio and final drive, splitting it
   across the driven wheels ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
4. **The wheels** (stage 5) receive drive torque; each computes a longitudinal tyre force from its slip ratio and
   load (stage 6).
5. **The tyre forces** push the body forward; `IntegrateMotion` (stage 7,
   [C39.4](04-integrate.md)) integrates them into acceleration, and the car **speeds up** (stage 8).

So a throttle press becomes forward motion through five transformations, each a stage of the pipeline. The same
tracing works for steering (→ lateral tyre force → yaw) and braking (→ negative longitudinal force → deceleration).
This is the value of the end-to-end view: any input can be followed to its effect on the car's motion.

## Why compose it this way

Building the car as a chain of mechanics (rather than one monolithic update) is a deliberate design
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)):

- **Swappable behaviour.** A cop car ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md) — `RBCop`)
  swaps the AI mechanic for a pursuit brain but keeps the same engine/suspension/rigidbody; the player swaps in
  the input mechanic. One chassis model, different drivers.
- **Data-driven tuning.** Each mechanic reads its parameters from the vault
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) — the engine's power curve, the suspension's
  stiffness, the tyre's grip — so a car is tuned by data, not code
  ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).
- **Clear boundaries.** Each stage has a defined input and output, so the pipeline is legible and the sim is
  isolated ([C39.5](05-connectors.md)).

So the input-to-tyres chain is the composition of eight tunable, swappable mechanics into one deterministic
frame — the architecture that lets Most Wanted have dozens of distinct cars and a shared, reliable physics.

## RE implications

- **A frame is a chain:** input/AI → INPUT/AI mechanic → ENGINE → drivetrain → wheels → suspension+tyres →
  RigidBody integrate → moved car.
- **The eight `BEHAVIOR_MECHANIC_*` are the stages** ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)),
  run in order by `Physics::Simulate`.
- **Any input can be traced** through the chain to the car's motion (throttle → force → acceleration).
- **Composition buys** swappable drivers, data-driven tuning, and clear, isolated boundaries.

---

### Key takeaways

- One frame of a car is a chain: **input/AI → AI/INPUT mechanic → engine → drivetrain → wheels → suspension+tyres
  → RigidBody integrate → move**.
- The **eight `BEHAVIOR_MECHANIC_*`** are the stages, run in order by `Physics::Simulate`
  ([C39.2](02-simulate.md)).
- Any input (throttle, steer, brake) can be **traced end to end** to its effect on the car's motion.
- The composition is deliberate: **swappable** drivers (player vs. cop), **data-driven** tuning, **isolated**
  boundaries.
- This architecture is what lets Most Wanted ship many distinct cars on one shared, deterministic physics core.

**Continue:** [C39.7 — Reading the vehicle sim in RE](07-reading-sim.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md)
