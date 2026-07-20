# C42.4 — The Tyre Model

> **The one-sentence version:** the tyre model is where each wheel turns its **load** (from suspension), its
> **slip** (ratio for drive/brake, angle for cornering), and the **surface grip** into the longitudinal and
> lateral forces the rigid body integrates — and tyres can be damaged (`ETireBlown`, `ETirePunctured`).

[← C42.3 — Suspension](03-suspension.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md) ·
[Next: C42.5 — The tuning surface →](05-tuning-surface.md)

---

## Where rubber meets road

The tyre model is the final transformation in the drive chain
([C39.6](../C39-Vehicle-Simulation/06-input-to-tyres.md)): everything upstream — engine torque
([C42.2](02-engine-drivetrain.md)), wheel loads ([C42.3](03-suspension.md)) — becomes an actual **force on the
body** only through the tyre's contact patch. Each wheel computes two forces:

- **Longitudinal force** — drive or brake, along the direction the wheel rolls. From the **slip ratio** (how much
  faster/slower the tyre surface moves than the road — spinning under power, locked under braking).
- **Lateral force** — cornering, perpendicular to the roll direction. From the **slip angle** (the angle between
  where the wheel points and where it's actually going).

Both forces are limited by the tyre's **grip**, which scales with the wheel **load** (from the suspension,
[C42.3](03-suspension.md)) and the **surface** ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)). This is
the classic tyre model: force rises with slip up to a peak, then falls off as the tyre breaks loose.

> 🟡 *Reasoned:* the slip-ratio/slip-angle tyre-force model with load and surface scaling is the standard vehicle
> tyre simulation, consistent with the verified per-wheel part array
> ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)), the suspension load model ([C42.3](03-suspension.md)),
> and the surface-grip read ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)); the exact tyre-force
> formula and its coefficients are per-car vault tunables and deeper RE. The tyre-damage states are verified
> strings.

## The friction circle

The two forces share one grip budget — the **friction circle**. A tyre can produce longitudinal force *or*
lateral force *or* a combination, but the *total* can't exceed what the grip (load × surface) allows:

- **Full braking** uses the grip budget longitudinally — little left for cornering (why you can't brake hard and
  turn hard at once).
- **Full cornering** uses it laterally — little left for accelerating (why power-on mid-corner can break the
  rear loose — spin).
- **Trail-braking / balanced throttle** blends the two within the circle — the skill of fast driving is staying
  at the edge of the circle without exceeding it.

This coupling is what makes the driving model feel real: weight transfer ([C42.3](03-suspension.md)) changes each
tyre's load (its circle size), and the driver's inputs spend each circle's budget between drive/brake and
cornering. The interplay of four friction circles, fed by weight transfer, *is* the handling.

## Tyre damage: blown and punctured

Tyres are not indestructible — the verified states **`ETireBlown`** and **`ETirePunctured`** are tyre-damage
outcomes:

- **`ETirePunctured`** — a tyre losing pressure, typically from a **spike strip**
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) — the police roadblock tool
  that deflates tyres to slow a fleeing car.
- **`ETireBlown`** — a fully destroyed tyre — worse grip, pulling to one side, a crippled car.

A damaged tyre has drastically reduced grip (a smaller friction circle), so the car handles poorly — exactly the
effect a spike strip is meant to have in a pursuit. That these are enumerated states (`E*` = enum) means the tyre
model has a discrete damage dimension on top of the continuous slip/load model: a tyre is `normal`, `punctured`,
or `blown`, and its grip is scaled accordingly.

> ✅ *Verified:* `ETireBlown` and `ETirePunctured` are present as strings in `speed.exe` — enumerated tyre-damage
> states. They connect the tyre model to the pursuit spike-strip mechanic
> ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).

## Tyre modes and the surface

While driving, the tyre is always in a **mode** — what it's doing — and the mode (crossed with the surface) drives
the presentation ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)):

- **driving** — rolling normally (grip in the linear range).
- **skid** — braking/locked (longitudinal slip past the peak).
- **slide** — drifting (lateral slip past the peak).
- **fly / hit** — airborne / landing.

The mode is a readout of the tyre's slip state, and it's what the sound (`RoadNoiseRecord`) and effect
(`TireEffectRecord`) systems key off ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) — a slide on
asphalt sounds and smokes differently than a skid on gravel. So the tyre model produces not just forces but a
*mode*, and that mode is the bridge to the car's audiovisual feedback (the screech, the smoke). The tyre is where
physics and presentation meet at the contact patch.

## RE implications

- **The tyre model turns load + slip + surface grip into force** — longitudinal (slip ratio) and lateral (slip
  angle), the forces the body integrates.
- **The friction circle** couples the two — total force is bounded by grip; driving skill is spending the budget.
- **Tyres can be damaged** — `ETirePunctured` (spike strip) and `ETireBlown` (destroyed) reduce grip; verified
  states.
- **The tyre mode** (driving/skid/slide/fly/hit) bridges to sound and effects
  ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)).

---

### Key takeaways

- The tyre model is where **load (suspension) + slip + surface grip** become the **longitudinal and lateral
  forces** the rigid body integrates — the final drive-chain transformation.
- The **friction circle** couples drive/brake and cornering into one grip budget — you can't brake hard and turn
  hard at once; fast driving lives at the circle's edge.
- **Tyres can be damaged** — verified states `ETirePunctured` (spike strips) and `ETireBlown` (destroyed) shrink
  the friction circle.
- The tyre's **mode** (driving/skid/slide/fly/hit) is the readout that drives the car's sound and smoke
  ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)).
- The contact patch is where **physics and presentation meet** — force, and feedback, from four tyres.

**Continue:** [C42.5 — The tuning surface](05-tuning-surface.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md)
