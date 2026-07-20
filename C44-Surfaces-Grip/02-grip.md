# C44.2 — Grip: the Functional Read

> **The one-sentence version:** the grip read is the one that changes *where the car goes* — the tyre model scales
> its available grip by the surface, so asphalt holds and sand/grass/ice slide, making the surface a real physical
> condition, not just decoration.

[← C44.1 — The surface taxonomy](01-surface-taxonomy.md) · [Chapter 44 hub](C44-Surfaces-Grip.md) ·
[Next: C44.3 — RoadNoiseRecord →](03-road-noise.md)

---

## The functional read

Of the three reads of a surface tag ([C44.5](05-three-reads.md)), **grip** is the *functional* one — the only one
that changes the car's trajectory. The other two (sound, effects) are presentation; grip is physics. The tyre
model ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) uses the wheel's surface tag to **scale
the available grip** at that contact patch:

- **High-grip surfaces** (`asphalt`, `concrete`) — the tyre holds; you can brake hard, corner hard, put power
  down. Full friction circle ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)).
- **Low-grip surfaces** (`grass`, `sand`, `dirt`) — the tyre slides; braking is longer, cornering washes out,
  power spins the wheels. Shrunken friction circle.
- **Very-low-grip surfaces** (`ice`, `water`, `snow`) — the tyre barely holds; the car slithers, understeers
  badly, and is hard to control.

So driving off the road onto grass isn't cosmetic — the grass *physically reduces your grip*, and you feel the car
wash wide. This is what makes the world's surfaces matter to driving: the racing line is partly about staying on
the high-grip surface.

> 🟡 *Reasoned:* the surface-scales-grip model (a per-surface grip coefficient multiplying the tyre's friction) is
> the standard surface-friction design, consistent with the verified surface tags ([C44.1](01-surface-taxonomy.md))
> and the tyre model ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)); the exact per-surface
> grip coefficients live in the tyre/suspension tunables and aren't enumerated here. The surface tags and their
> presence are verified.

## Per-wheel grip

Because the surface is read **per wheel** ([C44.1](01-surface-taxonomy.md)), grip is per-wheel too — and this
produces rich, believable behaviour when a car straddles a surface boundary:

- **Two wheels on, two off.** Dropping the right-side wheels onto grass while the left stays on asphalt gives
  *asymmetric* grip — the car pulls toward the low-grip side, and can spin if you're not careful. The classic
  "dropped a wheel off the track" moment.
- **Rear on grass, front on road (or vice versa).** Losing grip at one axle but not the other shifts the car's
  balance toward oversteer or understeer momentarily — exiting a corner over a grassy apex can snap the rear.

So the per-wheel surface read means the car responds to *exactly* which wheels are on what — not a single "the car
is on grass" flag. This granularity is why surface transitions feel physical: the car reacts continuously as each
tyre crosses each boundary. It's the same per-wheel structure as the suspension loads
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)) — four independent contact patches, each with
its own surface and grip.

## Grip and the friction circle

Grip is the surface's input to the **friction circle** ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)):
the circle's *size* is set by the wheel load (suspension, [C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md))
*and* the surface grip. A low-grip surface shrinks the circle, so there's less total force available to split
between braking, cornering, and acceleration:

- **On ice**, the circle is tiny — any significant brake, steer, or throttle input exceeds it and the tyre slides.
  This is why ice feels uncontrollable: you have almost no force budget.
- **On asphalt**, the circle is full — you have the whole budget to spend, so the car is responsive and
  controllable.

So the surface, through grip, sets *how much* the driver can ask of the tyres. The skill of driving across mixed
surfaces is managing a *changing* friction circle — easing off when a wheel drops onto grass, because its circle
just shrank. Grip is the surface's hand on the car's limits.

## Why grip is code-side data

The grip coefficient is *data* (a per-surface tunable, [Chapter 42](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md))
consumed by *code* (the tyre model) — the same split as everything else
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)):

- **The tyre model (code)** knows how to turn grip × load × slip into force — fixed physics.
- **The per-surface grip (data)** is a number per surface — tunable, so designers can make grass more or less
  slippery without touching the tyre code.

So grip sits on the boundary: it's the surface's *physical* effect, but authored as a *number*. A mod that makes
off-road driving more forgiving raises the grip coefficients for grass/dirt; the tyre model does the rest. This is
the functional read's version of the tuning surface ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md))
— the physics is fixed, the per-surface feel is data.

## RE implications

- **Grip is the functional read** — the surface scales the tyre's available grip, changing where the car goes
  (the other reads are presentation).
- **High/low/very-low grip** — asphalt/concrete hold; grass/sand/dirt slide; ice/water/snow slither.
- **Grip is per-wheel** — straddling a surface boundary gives asymmetric grip (pull, oversteer/understeer).
- **Grip sizes the friction circle** — low grip = tiny force budget (ice); the surface sets the car's limits.

---

### Key takeaways

- **Grip** is the one **functional** read of the surface — it changes the car's trajectory (sound and effects are
  presentation).
- The tyre model **scales grip by surface**: asphalt/concrete hold, grass/sand/dirt slide, ice/water/snow
  slither — so leaving the road physically loosens the car.
- Grip is **per-wheel** — straddling a boundary gives **asymmetric grip** (the car pulls, or the balance shifts to
  over/understeer).
- Grip sizes the **friction circle** ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) — low grip
  shrinks the force budget (ice feels uncontrollable); driving mixed surfaces is managing a changing circle.
- Grip is a **per-surface tunable** consumed by the fixed tyre code — the functional read's tuning surface.

**Continue:** [C44.3 — RoadNoiseRecord: the sound](03-road-noise.md) · [Chapter 44 hub](C44-Surfaces-Grip.md)
