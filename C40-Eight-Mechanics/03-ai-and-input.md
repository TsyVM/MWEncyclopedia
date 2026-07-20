# C40.3 — AI & INPUT: The Two Drivers

> **The one-sentence version:** two mechanics produce a car's controls — `BEHAVIOR_MECHANIC_INPUT` reads the
> player's pad/wheel, `BEHAVIOR_MECHANIC_AI` runs the AI driver for cops and traffic — and a car uses whichever
> is active, over an identical rest-of-car.

[← C40.2 — RIGIDBODY](02-rigidbody.md) · [Chapter 40 hub](C40-Eight-Mechanics.md) ·
[Next: C40.4 — ENGINE →](04-engine.md)

---

## Two mechanics, one job

The eight mechanics include **two drivers** — `BEHAVIOR_MECHANIC_INPUT` and `BEHAVIOR_MECHANIC_AI` — and they do
the same job from different sources: produce the car's **controls** (throttle 0..1, brake 0..1, steering −1..1,
handbrake, gear). These controls are the sim's input ([C39.6](../C39-Vehicle-Simulation/06-input-to-tyres.md)):
the engine mechanic ([C40.4](04-engine.md)) reads throttle, the steering feeds the front wheels, the brake feeds
all wheels.

That the schema has *both* as mechanics ([C40.1](01-the-mechanic-model.md)) is the elegant core of the design:
the source of the controls is swappable, and everything downstream (engine, suspension, rigid body) is identical
regardless of who's driving.

## INPUT: the player

`BEHAVIOR_MECHANIC_INPUT` is the **player's** driver: it reads the physical input device (gamepad, wheel,
keyboard) and maps it to controls. Its job is translation and feel:

- **Device → controls.** The right trigger becomes throttle, the left trigger brake, the stick steering
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md) reads the device each frame).
- **Response shaping.** The raw input is shaped — deadzones, sensitivity curves, steering speed — so the car
  feels right, tuned by input parameters ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
- **Assists.** Any driver assists (braking assist, steering aid) live here, between the device and the controls.

So the input mechanic is the player's hands on the car — the layer that turns button presses into the same
control values the AI produces.

## AI: cops and traffic

`BEHAVIOR_MECHANIC_AI` is the **AI** driver: instead of a device, it runs the AI driver brain
([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) to *decide* the controls. Its job is judgement:

- **Goal-driven.** The AI has goals (chase, flee, patrol, [Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md))
  that determine where it wants to go.
- **Path following.** It follows the road network ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md))
  toward its target, computing the steering and throttle to stay on the road at speed.
- **Reaction.** It reacts to obstacles, the player, and other cars — braking, swerving, ramming
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).

The output, though, is the same shape as INPUT's: throttle, brake, steer. The AI is a driver that produces
controls by thinking rather than by reading a pad.

> ✅ *Verified:* `BEHAVIOR_MECHANIC_AI` and `BEHAVIOR_MECHANIC_INPUT` are both present in `attributes.bin` as
> distinct mechanics ([C40.1](01-the-mechanic-model.md)). The AI driver brain and goal/action system are detailed
> in [Chapters 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)–[47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md).

## The swap: player becomes cop

Because the driver is a swappable mechanic, a profound simplification follows: **a cop car is a player car with
the AI mechanic instead of input.** The chassis, engine, suspension, and rigid body
([C40.2](02-rigidbody.md)–[C40.5](05-suspension.md)) are the same components; only the driver mechanic differs.
This is why:

- **Cops drive like real cars.** They obey the same physics — a cop can spin out, understeer, or crash, because
  it's the same rigid body and tyres as the player, just driven by AI.
- **The RBCop vs. RBVehicle distinction** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) is
  mostly about *which driver and behaviour*, not a different physics.
- **Traffic is the same.** Ambient traffic ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) is
  cars with an AI driver and simpler goals.

So the two-driver design is what unifies the player and the world: everything on the road is the same kind of
car, differing only in who holds the wheel. This is a deliberate and powerful architectural choice
([C40.7](07-reading-mechanics.md)).

> 🟡 *Reasoned:* that a cop is a player-equivalent car with the AI driver mechanic swapped in is the direct
> consequence of the verified two-driver-mechanic design and the shared rigid-body hierarchy
> ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)); the per-car AI parameter differences are
> vault data.

## RE implications

- **Two driver mechanics** — `INPUT` (player device → controls) and `AI` (AI brain → controls) — produce the same
  control shape.
- **INPUT** shapes device input (deadzones, sensitivity, assists); **AI** decides controls via goals + path
  following ([Chapters 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)–47).
- **A cop is a player car with the AI mechanic** — same physics, different driver.
- **The swap unifies player and world** — everything on the road is the same car, differing by driver.

---

### Key takeaways

- Two mechanics produce a car's **controls**: `BEHAVIOR_MECHANIC_INPUT` (player device) and
  `BEHAVIOR_MECHANIC_AI` (AI brain) — same output shape (throttle/brake/steer).
- **INPUT** translates and shapes the physical device; **AI** decides controls via goals and path following.
- Because the driver is a **swappable mechanic**, a cop is a player car with the **AI mechanic** in place of
  input — identical physics.
- This two-driver design **unifies the player and the world** — every car on the road is the same kind, differing
  only by who drives.
- Downstream mechanics (engine, suspension, rigid body) are **identical regardless** of driver.

**Continue:** [C40.4 — ENGINE: power & drivetrain](04-engine.md) · [Chapter 40 hub](C40-Eight-Mechanics.md)
