# C40.4 — ENGINE: Power & Drivetrain

> **The one-sentence version:** `BEHAVIOR_MECHANIC_ENGINE` is the powertrain — it turns the driver's throttle
> into wheel torque through an RPM/power curve and the gearbox, and its parameters are what make one car quicker
> than another.

[← C40.3 — AI & INPUT](03-ai-and-input.md) · [Chapter 40 hub](C40-Eight-Mechanics.md) ·
[Next: C40.5 — SUSPENSION →](05-suspension.md)

---

## Throttle to wheel torque

`BEHAVIOR_MECHANIC_ENGINE` is the mechanic that makes a car go. Its job is to convert the driver's **throttle**
([C40.3](03-ai-and-input.md)) into **torque at the driven wheels**, through the powertrain model:

```
throttle + current RPM → engine torque (power curve)
   → × gear ratio → × final drive → wheel torque
```

The engine produces torque as a function of RPM (the power curve — peaky for a race engine, broad for a muscle
car), the gearbox multiplies it by the current gear's ratio, and the final drive multiplies again, delivering
torque to the driven wheels ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
That wheel torque becomes a longitudinal tyre force ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md))
that accelerates the car.

## The RPM loop

The engine and wheels are coupled in a loop through RPM:

- **Wheel speed → RPM.** The driven wheels' rotation, back through the drivetrain, sets the engine RPM (in gear).
- **RPM → torque.** The RPM indexes the power curve for available torque.
- **Torque → wheel force → acceleration → faster wheels → higher RPM.**

This loop is what produces the feel of an engine: revving through the power band, hitting the redline, shifting
(which changes the gear ratio, dropping RPM), and the torque dip between gears. The engine mechanic runs this
loop each frame, and it's tightly tied to the **engine sound** ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)):
the GIN synth's RPM input is this same engine RPM, which is why the sound tracks the simulation exactly (the
`Gnsu` header's rpmMin/rpmMax, [C22.2](../C22-Engine-Sound-GIN/02-gnsu-header.md), bound the audible range).

> ✅ *Verified:* the engine sound (GIN) is driven by engine RPM, with the `Gnsu` header carrying rpmMin at +0x08
> (2267.0) and rpmMax at +0x0C (8638.1) ([C22.2](../C22-Engine-Sound-GIN/02-gnsu-header.md)) — the RPM range the
> engine mechanic sweeps. `BEHAVIOR_MECHANIC_ENGINE` is present in `attributes.bin`
> ([C40.1](01-the-mechanic-model.md)).

## Parameters make the car

The engine mechanic is where a car's *character* as a performance object lives, and it's entirely data-driven
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)):

- **Power curve** — torque vs. RPM: the shape (peaky vs. flat) and peak (how much power).
- **Redline** — the max RPM, where power falls off and shifting is forced.
- **Gear ratios** — the number of gears and each ratio: close ratios for acceleration, tall top gear for speed.
- **Final drive** — the overall gearing multiplier.

Tuning these ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) is what upgrades do — a performance
package raises the power curve and adjusts the gearing. Two cars with the same chassis but different engine
parameters accelerate and top out differently. So the engine mechanic is the primary lever of a car's
performance identity ([C40.7](07-reading-mechanics.md)).

## Forced induction and its sound

Many Most Wanted cars have forced induction (turbo/supercharger), and the engine mechanic models it as an
addition to the power curve (boost that builds with RPM/load). This too has an audible signature — the turbo
spool and blow-off ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) are driven by the engine
mechanic's boost state, one more case of the sound tracking the sim. The engine mechanic is thus the source of
both the *acceleration* and (through the sound mechanic, [C40.6](06-damage-draw-sound.md)) the *aural drama* of a
car.

> 🟡 *Reasoned:* the power-curve/gearing/forced-induction structure of the engine mechanic is the standard
> vehicle-powertrain model, consistent with the verified engine-RPM→GIN-synth binding
> ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) and the car-tuning vault categories
> ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); the exact parameter fields are per-car vault
> data.

## RE implications

- **`BEHAVIOR_MECHANIC_ENGINE`** converts throttle + RPM → engine torque → (× gearing) → wheel torque.
- **The RPM loop** couples engine and wheels — revving, redline, shifting — and drives the engine sound
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).
- **Its parameters make the car** — power curve, redline, gear ratios, final drive — all vault-tuned
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
- **It's the source of performance identity** — upgrades tune these parameters.

---

### Key takeaways

- `BEHAVIOR_MECHANIC_ENGINE` is the **powertrain** — throttle + RPM → engine torque → gearbox → **wheel torque**.
- The **RPM loop** (wheel speed ↔ engine RPM ↔ torque) produces revving, redline, and shifting — and drives the
  **engine sound** ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md), verified `Gnsu` rpm range).
- The engine's **parameters** (power curve, redline, gears, final drive) are what make one car quicker than
  another — all vault data.
- Upgrades and forced induction are **parameter changes** to this mechanic.
- The engine mechanic is the primary lever of a car's **performance identity**.

**Continue:** [C40.5 — SUSPENSION: loads & weight transfer](05-suspension.md) · [Chapter 40 hub](C40-Eight-Mechanics.md)
