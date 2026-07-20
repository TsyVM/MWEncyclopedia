# C45.1 — The Damage Mechanic Family

> **The one-sentence version:** damage is a mechanic family — `DamageVehicle` (vtable `0x008AD288`, **127
> methods**, the base) plus per-role tiers `DamageRacer` (98), `DamageCopCar` (36), `DamageHeli` (65) — each a
> verified vtable, tuned by `DamageParams`.

[← Chapter 45 hub](C45-Damage-Deformation.md) · [Next: C45.2 — Damage zones →](02-damage-zones.md)

---

## The family

Like the engine and suspension ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)),
the `BEHAVIOR_MECHANIC_DAMAGE` slot ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) is a **family of
implementations**, one per role, all verified vtables in `speed.exe`:

| Class | Hash | vtable | Methods | Role |
|---|---|---|---|---|
| `DamageVehicle` | `0x7A0C70A5` | `0x008AD288` | **127** | the base damage model |
| `DamageRacer` | `0x6AE5E09C` | `0x008AD2FC` | 98 | hero/racer cars (the player) |
| `DamageCopCar` | `0x1DF44901` | `0x008AD3F4` | 36 | cop cruisers ([C45.5](05-cop-damage.md)) |
| `DamageHeli` | `0x4599C96A` | `0x008AD380` | 65 | the police helicopter |

`DamageVehicle`'s **127 methods** make it the *most-detailed* class in the whole vehicle mechanic set — more than
`EngineRacer` (123, [C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)). Deformation is genuinely
intricate: health per zone, visual crumple, breakable parts, and performance penalties all live here.

> ✅ *Verified:* the `Damage*` vtables and method counts are confirmed in `speed.exe` by counting consecutive code
> pointers — `DamageVehicle` `0x008AD288`/127, `DamageRacer` `0x008AD2FC`/98, `DamageCopCar` `0x008AD3F4`/36,
> `DamageHeli` `0x008AD380`/65. `rh("DamageVehicle")=0x7A0C70A5` ×1 and `rh("DamageRacer")=0x6AE5E09C` ×3 are vault
> keys; the constructor references `DamageParams` (`0x7BA51D5C`).

## Why 127 methods

`DamageVehicle` being the most-methoded class reflects how much a damage model must do — far more than the "just
health" one might expect:

- **Per-zone accounting.** Track health/deformation for each of the ~6 coarse zones ([C45.2](02-damage-zones.md))
  and each part-specific breakable — many small methods per zone/part.
- **Visual deformation.** Progressively crumple the mesh ([C45.3](03-deformation.md)) as a zone accumulates
  damage — geometry manipulation.
- **Breakables.** Switch each part (bumper, hood, glass, [C45.3](03-deformation.md)) between intact and
  broken/detached states.
- **Performance penalties.** Degrade engine/handling as damage mounts ([C45.4](04-scaling-performance.md)) —
  feedback into the other mechanics.
- **Reset.** Restore the car (respawn, [ResetCar](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) to
  undamaged.

Each of these is several methods, per zone and per part — so a fully-featured damage model naturally has the most
methods. The 127 is the measure of how seriously Most Wanted takes visible, consequential damage.

## Per-role fidelity

The tiers mirror the engine/suspension pattern ([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) —
different roles need different damage detail:

- **`DamageRacer` (98).** The full player experience: visible crumple, part detachment, and handling degradation
  the player sees and feels ([C45.4](04-scaling-performance.md)). Nearly as detailed as the base.
- **`DamageCopCar` (36 — the fewest).** Cops take damage and get *disabled* ([C45.5](05-cop-damage.md)), but don't
  need the player's fine-grained crumple — 36 methods for "damage, then totalled," plus the cop-specific parts
  (light bar, [C45.5](05-cop-damage.md)).
- **`DamageHeli` (65).** The chopper's disable/destruction model ([C41.6](../C41-Physics-RigidBody/06-vehicle-types.md))
  — it can be lost in some modes.

So damage fidelity is paid where it's seen: the player's car (`DamageRacer`) gets the full model, the cop
(`DamageCopCar`) gets a coarser "disable" model, and the chopper a middle one. This is physics-LOD
([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) applied to damage — full crumple for the car
you're looking at, "disabled" for the cars you're escaping.

## DamageParams: the schema

`DamageVehicle`'s constructor references **`DamageParams`** — the base parameter class ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md))
defining the tunable damage fields: the per-zone health thresholds, the deformation limits, the performance-penalty
curves. Each car's vault data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) supplies the values.
So, as everywhere ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)), the `Damage*` class is the
code and `DamageParams`-shaped vault data is the configuration — a car's toughness is data, its damage *behaviour*
is the shared class.

## RE implications

- **Damage is a mechanic family** — `DamageVehicle` (127, base), `DamageRacer` (98), `DamageCopCar` (36),
  `DamageHeli` (65) — verified vtables.
- **`DamageVehicle`'s 127 methods** are the most in the vehicle mechanic set — per-zone accounting, deformation,
  breakables, penalties, reset.
- **Per-role fidelity** — full crumple for the player (`DamageRacer`), coarse disable for cops (`DamageCopCar`).
- **`DamageParams`** is the schema — the car's toughness is vault data, the behaviour is the shared class.

---

### Key takeaways

- Damage is a **mechanic family**: `DamageVehicle` (base, **127 methods**), `DamageRacer` (98, player),
  `DamageCopCar` (36, cop), `DamageHeli` (65) — all **verified vtables**.
- `DamageVehicle` is the **most-methoded** vehicle mechanic — deformation is intricate (per-zone health, crumple,
  breakables, performance penalties, reset).
- **Per-role fidelity** pays detail where it's seen — full crumple for the player, coarse "disable" for cops
  (fewest methods).
- The constructor references **`DamageParams`** — a car's toughness is **vault data**, its damage behaviour is the
  **shared class**.
- Damage sits in the `BEHAVIOR_MECHANIC_DAMAGE` slot ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md))
  — one of the eight, reading the collisions the body detects.

**Continue:** [C45.2 — Damage zones](02-damage-zones.md) · [Chapter 45 hub](C45-Damage-Deformation.md)
