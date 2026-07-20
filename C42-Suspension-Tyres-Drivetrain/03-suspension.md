# C42.3 — Suspension

> **The one-sentence version:** `SuspensionRacer` (vtable `0x008ABAC0`, 45 methods) is the full suspension —
> per-wheel spring/damper, ride height, anti-roll, and load transfer — the model that makes a tuned car feel
> planted or loose, and its constructor references the `SuspensionParams` base.

[← C42.2 — The engine & drivetrain](02-engine-drivetrain.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md) ·
[Next: C42.4 — The tyre model →](04-tyres-grip.md)

---

## The full suspension

`SuspensionRacer` (verified vtable `0x008ABAC0`, 45 methods) is the hero-car suspension — the model that computes
each wheel's load and the car's weight transfer ([C40.5](../C40-Eight-Mechanics/05-suspension.md)). Its elements:

- **Per-wheel spring/damper.** Each of the four wheels has a spring (force ∝ compression) and a damper (force ∝
  compression *rate*), computing the vertical force at that corner.
- **Ride height.** The static compression at rest — how low the car sits, and the starting point for travel.
- **Anti-roll bars.** Front and rear bars that resist body roll by coupling the left/right wheels — stiffer bars
  reduce lean and shift the understeer/oversteer balance.
- **Load transfer.** The redistribution of the four wheel loads under acceleration, braking, and cornering
  ([C40.5](../C40-Eight-Mechanics/05-suspension.md)) — the physical basis of the car's balance.

The output, each frame, is four wheel loads — the grip budget the tyre model ([C42.4](04-tyres-grip.md)) turns
into cornering and traction forces.

> ✅ *Verified:* `SuspensionRacer` is a real vtable at `0x008ABAC0` with 45 methods; its hash `0x6209E06A`
> appears ×3 as a vault key in `attributes.bin`. The family includes `SuspensionSimple` (44, `0x008ABC28`),
> `SuspensionTraffic` (86, `0x008ABB80`), `SuspensionSpline` (57, `0x008ABD88`), `SuspensionTrailer` (99,
> `0x008ABCE0`). The constructor references the `SuspensionParams` base class.

## SuspensionParams: the base

`SuspensionRacer`'s constructor references **`SuspensionParams`** — the base parameter class that defines the
*shape* of a suspension's tunables. This is the reflection pattern ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)):
`SuspensionParams` is the schema (what fields a suspension has — spring rate, damper rate, ride height, anti-roll,
travel), and each car's vault collection ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) supplies
the *values*. So `SuspensionRacer` (the code) reads `SuspensionParams`-shaped data (the numbers) to configure this
car's suspension. The class consumes the schema; the vault fills it in.

> 🟡 *Reasoned:* the specific suspension elements (spring/damper/anti-roll/ride-height/load-transfer) are the
> standard vehicle-suspension model, consistent with `SuspensionRacer`'s verified vtable and its
> `SuspensionParams` reference; the exact field set and per-method behaviour are deeper RE. The class, vtable,
> method count, hash, and `SuspensionParams` reference are verified.

## Handling character, computed

The suspension is where a car's **handling character** is computed ([C40.5](../C40-Eight-Mechanics/05-suspension.md)) —
and the parameters make the feel:

- **Spring stiffness.** Stiff = flat, responsive, less weight transfer (a race car); soft = more roll and
  transfer, more forgiving (a cruiser). Front/rear balance of stiffness tilts toward understeer or oversteer.
- **Damping.** Controls how fast load changes settle — too soft floats and wallows, too stiff skips over bumps
  and loses grip.
- **Ride height.** Lower generally means less weight transfer (lower centre of gravity) and a more planted feel.
- **Anti-roll.** More front anti-roll → more understeer; more rear → more oversteer. The classic balance knob.

Because these are all vault fields ([C42.5](05-tuning-surface.md)), two cars with identical engines can handle
completely differently, and a suspension tune reshapes a car's feel without touching a line of code. The
`SuspensionRacer` class is the *fixed* physics; the *variable* is the `SuspensionParams` data.

## Why traffic has more methods

As noted ([C42.1](01-fidelity-tiers.md)), `SuspensionTraffic` has **86** methods to `SuspensionRacer`'s 45 — the
"cheap" tier has nearly double. The resolution: `SuspensionRacer` does a *thorough* per-wheel solve in relatively
few, heavy methods; `SuspensionTraffic` does a *cheap* approximation, but built from many small specialised
methods (per-wheel, per-case) optimised to run fast across the whole traffic population. Fewer heavy methods
(racer) vs. many light methods (traffic) — the *cost per car* favours traffic despite the higher count. And
`SuspensionTrailer` (99, the most) is genuinely the hardest: a multi-axle towed box with a swinging hitch
([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) needs the most stabilisation logic. So method count is a
window onto each tier's *strategy*, not a simple fidelity ranking.

## RE implications

- **`SuspensionRacer` (45 methods, `0x008ABAC0`)** is the full suspension — per-wheel spring/damper, ride height,
  anti-roll, load transfer.
- **It references `SuspensionParams`** — the schema; the car's vault supplies the values
  ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)).
- **Handling character is computed here** — stiffness, damping, ride height, anti-roll — all vault-tuned.
- **Method counts reflect strategy** — racer (few heavy methods) vs. traffic (many light) vs. trailer (99, the
  hardest).

---

### Key takeaways

- `SuspensionRacer` (**verified** vtable `0x008ABAC0`, 45 methods) is the **full suspension** — per-wheel
  spring/damper, ride height, anti-roll bars, and load transfer.
- Its constructor references **`SuspensionParams`** — the schema class; the per-car vault fills in the values.
- The suspension computes a car's **handling character** (stiffness, damping, ride height, anti-roll balance) —
  all vault-tuned, so cars with identical engines can feel completely different.
- Its output is **four wheel loads** — the grip budget the tyre model ([C42.4](04-tyres-grip.md)) turns into
  force.
- **Method counts reflect strategy**: `SuspensionRacer` (45, few heavy), `SuspensionTraffic` (86, many light),
  `SuspensionTrailer` (99, the hardest to stabilise).

**Continue:** [C42.4 — The tyre model](04-tyres-grip.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md)
