# C42.5 — The Tuning Surface

> **The one-sentence version:** everything in the drivetrain and suspension is tunable by data — gear ratios,
> torque, NOS, spring rates, ride height, anti-roll, mass transfer are all vault fields in the per-car
> collections — so the `Engine*`/`Suspension*` classes are the code that consumes numbers, and a performance mod
> edits the numbers, not the classes.

[← C42.4 — The tyre model](04-tyres-grip.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md) ·
[Next: C42.6 — Reading the drivetrain in RE →](06-reading-drivetrain.md)

---

## Code consumes, data configures

The central fact of car tuning is a clean separation ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)):

- **The classes are fixed code.** `EngineRacer` ([C42.2](02-engine-drivetrain.md)) and `SuspensionRacer`
  ([C42.3](03-suspension.md)) are the *physics* — the torque-curve evaluation, the gearbox logic, the spring/damper
  solve. Every car uses the same code.
- **The parameters are per-car data.** The *numbers* those classes read — gear ratios, peak torque, NOS bonus,
  spring rate, ride height, anti-roll stiffness — live in the car's vault collection
  ([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)), keyed by the reflection hash
  ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)).

So a car *is* its parameters, run through the shared drivetrain/suspension code. This is why the garage can offer
deep tuning without touching the engine's C++: tuning edits the vault data the classes consume.

> ✅ *Verified:* the drivetrain/suspension specs are **vault keys** in `GLOBAL/attributes.bin` — `EngineRacer`
> (`0xB2809518`) ×4, `EngineTraffic` (`0x5C216BAB`) ×2, `SuspensionRacer` (`0x6209E06A`) ×3, `SuspensionTraffic`
> (`0x12D5313C`) ×2, `SuspensionSimple` (`0x723B315B`) ×1, `SuspensionTrailer` (`0xD44C9372`) ×1. The classes read
> `*Params`-shaped vault data ([C42.3](03-suspension.md)).

## The tuning fields

The tunable surface spans both mechanics ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)):

| Mechanic | Tunable fields (vault) |
|---|---|
| Engine ([C42.2](02-engine-drivetrain.md)) | torque curve / peak power, gear ratios, final drive, NOS bonus & tank, redline |
| Suspension ([C42.3](03-suspension.md)) | spring rate, damper rate, ride height, anti-roll (front/rear), travel |
| Tyre ([C42.4](04-tyres-grip.md)) | grip coefficient, load sensitivity, slip response |
| Body ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) | mass, centre of gravity, inertia, drag |

Every one of these is a number in the vault, and every one changes how the car drives. A performance-parts upgrade
is, mechanically, a *change to these numbers* — a better turbo raises the torque curve, sport springs raise the
spring rate, a weight reduction lowers the mass. The garage UI ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md))
is a front-end onto vault edits.

## Why data-driven tuning matters

Making the entire driving model data-driven ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) is a
deep design decision with broad payoffs:

- **Designers tune without engineers.** A designer adjusts a car's feel by editing vault numbers, iterating in
  data — no recompile, no engineering time per tweak. The whole garage of cars is balanced this way.
- **Upgrades are data.** The performance-parts system ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
  is just staged parameter sets — stock vs. sport vs. pro are three number sets for the same classes.
- **One code path, all cars.** Because every car runs the same `EngineRacer`/`SuspensionRacer` code with different
  data, there's one physics to debug and verify — bugs are fixed once, for every car.
- **Modding surface.** A mod that changes handling edits the vault, not the executable — the classes are stable,
  the data is open ([C42.6](06-reading-drivetrain.md)).

So the tuning surface is where Most Wanted's driving depth and its engineering economy meet: a rich, tunable
driving model built once in code and expressed a hundred ways in data. This is the industry pattern of a
data-driven vehicle system, and MW's implementation — verified specs keyed into a reflection-hashed vault — is a
clean example of it.

## The editing rule

The practical rule for anyone modifying the driving model ([C42.6](06-reading-drivetrain.md)):

- **Edit the numbers, not the classes.** Gear ratios, spring rates, NOS, mass — all vault
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). Retuning a car is a vault edit.
- **The classes define what's tunable.** `SuspensionParams` ([C42.3](03-suspension.md)) is the schema — you can
  set the fields it defines, not invent new physics without new code.
- **Keep the tiers in mind.** A traffic car uses `EngineTraffic`/`SuspensionTraffic` ([C42.1](01-fidelity-tiers.md)) —
  a smaller tunable surface than a hero car's `*Racer`.

So the drivetrain/suspension chapter ends where car tuning begins: the classes here are the *engine* of tuning,
and the vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) is its *content*. Edit the content;
the engine does the rest.

## RE implications

- **Code consumes, data configures** — `Engine*`/`Suspension*` classes are fixed physics; the parameters are
  per-car vault data.
- **The tuning fields** span engine (torque, gears, NOS), suspension (springs, ride height, anti-roll), tyre
  (grip), and body (mass) — all vault.
- **Data-driven tuning** lets designers tune without engineers, makes upgrades data, and keeps one code path for
  all cars.
- **Edit the numbers, not the classes** — retuning is a vault edit; the classes define what's tunable.

---

### Key takeaways

- The drivetrain and suspension are **fully data-driven** — the `Engine*`/`Suspension*` **classes are the physics
  code**; the **parameters are per-car vault data** (verified specs as vault keys).
- The tunable surface spans **engine** (torque, gears, NOS), **suspension** (springs, ride height, anti-roll),
  **tyre** (grip), and **body** (mass) — every one a number in the vault.
- **A performance upgrade is a data change** — a better turbo raises the torque curve; sport springs raise the
  spring rate.
- Data-driven tuning gives **designer iteration without recompiles**, **upgrades as staged parameter sets**, and
  **one code path for all cars**.
- The rule: **edit the numbers, not the classes** — this chapter's classes are the *engine* of car tuning; the
  vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) is its *content*.

**Continue:** [C42.6 — Reading the drivetrain in RE](06-reading-drivetrain.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md)
