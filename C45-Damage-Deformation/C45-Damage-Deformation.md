# Chapter 45 — Damage & Deformation

> **Goal of this chapter:** decode how crashes hurt a car — the `Damage*` mechanic family (`DamageVehicle` base,
> 127 methods; `DamageRacer`, `DamageCopCar`, `DamageHeli`), the two damage-zone systems (coarse `DAMAGE0_*` and
> part-specific `DAMAGE_*` breakables), the `DamageScaleRecord` that scales a contact into damage, and the
> deformation and performance loss that result.

Damage is the *consequence* of the discrete side of touching the world
([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)): a contact is classified and reacts, and — in
parallel — it **hurts** the car. This chapter decodes the damage mechanic: how a car is divided into zones, how a
collision's force becomes zone damage (scaled by `DamageScaleRecord`), how that shows as visual deformation
(crumpled panels, broken glass, detached parts), and how it degrades performance. It's the system that makes a
crash *cost* something — visually and mechanically.

> **Verified against the executable and vault.** The `Damage*` mechanics are byte-verified vtables in `speed.exe`:
> `DamageVehicle` (hash `0x7A0C70A5`, vtable `0x008AD288`, **127 methods** — the base model), `DamageRacer`
> (`0x6AE5E09C`, `0x008AD2FC`, 98), `DamageCopCar` (`0x1DF44901`, `0x008AD3F4`, 36), `DamageHeli` (`0x4599C96A`,
> `0x008AD380`, 65). The **damage zones are strings** in the exe: coarse `DAMAGE0_FRONT/FRONTLEFT/FRONTRIGHT/REAR/
> REARLEFT/REARRIGHT`, and part-specific `DAMAGE_HOOD`, `DAMAGE_TRUNK`, `DAMAGE_FRONT_BUMPER`, `DAMAGE_LEFT_DOOR`,
> `DAMAGE_LEFT_HEADLIGHT`, `DAMAGE_FRONT_WINDOW`, `DAMAGE_COP_LIGHTS`, `DAMAGE_COP_SPOILER`, … plus glass detail
> (`LEFT_HEADLIGHT_GLASS`, `WINDSHIELD`, `BREAK_HEADLIGHT_LEFT`). **`DamageScaleRecord`** (`0xD99B853C`) is a vault
> key **×24** in `attributes.bin`; `DamageRacer` ×3.

---

## Deep-dive pages

- [C45.1 — The Damage mechanic family](01-damage-family.md): `DamageVehicle` (127) and the per-role tiers.
- [C45.2 — Damage zones](02-damage-zones.md): the coarse `DAMAGE0_*` and part-specific `DAMAGE_*` systems.
- [C45.3 — Deformation & breakables](03-deformation.md): crumpled panels, broken glass, detached parts.
- [C45.4 — Scaling & performance loss](04-scaling-performance.md): `DamageScaleRecord` (×24) and handling
  degradation.
- [C45.5 — Cop damage & the bust](05-cop-damage.md): `DamageCopCar`, the cop light bar, disabling cruisers.
- [C45.6 — Reading damage in RE](06-reading-damage.md): navigating the damage system.

---

## 45.1 The Damage family

Damage is a mechanic family ([C45.1](01-damage-family.md)): **`DamageVehicle`** is the base (127 methods — the
most-detailed of the whole vehicle mechanic set), modelling health/deformation per zone and the performance
penalties of a wrecked car. **`DamageRacer`** (98) is the hero-car model the player experiences; **`DamageCopCar`**
(36, the fewest) is the cop model — cruisers get disabled but don't need the player's fine crumple;
**`DamageHeli`** (65) is the chopper's. Its constructor references `DamageParams`.

## 45.2 Damage zones

A car is divided into **zones** ([C45.2](02-damage-zones.md)), and there are two systems: **coarse zones**
(`DAMAGE0_FRONT`, `_FRONTLEFT`, `_FRONTRIGHT`, `_REAR`, `_REARLEFT`, `_REARRIGHT` — six regions) that accumulate
impact energy, and **part-specific breakables** (`DAMAGE_HOOD`, `DAMAGE_FRONT_BUMPER`, `DAMAGE_LEFT_DOOR`,
`DAMAGE_LEFT_HEADLIGHT`, `DAMAGE_FRONT_WINDOW`, …) that break/detach individually. A contact's location decides
which zone and which parts take the hit.

## 45.3 Deformation & breakables

Damage shows as **deformation** ([C45.3](03-deformation.md)): panels crumple (the coarse zone's mesh deforms
progressively), glass breaks (`WINDSHIELD`, `LEFT_HEADLIGHT_GLASS`), and parts detach (bumpers, hoods). The
verified `BREAK_HEADLIGHT_LEFT`/`_RIGHT` and glass strings are the breakable-part system — each part has an intact
and a broken/removed state, switched when its zone takes enough damage.

## 45.4 Scaling & performance

How much a contact hurts is governed by **`DamageScaleRecord`** (×24, [C45.4](04-scaling-performance.md)) — it
scales the collision force ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) into zone damage,
per collision type. Enough damage causes **performance loss** — a wrecked engine loses power, a bent suspension
handles worse — feeding back into the other mechanics ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
`DamageScaleRecord` is separate from `CollisionReactionRecord` ([C43.4](../C43-Collision-Contacts/04-reactions.md)):
how the car is *hurt* vs. how it *moves*.

## 45.5 Cop damage & the bust

`DamageCopCar` (36 methods) is the cop damage model, and the verified `DAMAGE_COP_LIGHTS` and `DAMAGE_COP_SPOILER`
zones are cop-specific parts ([C45.5](05-cop-damage.md)) — the light bar you can knock off. Cops take damage and
get **disabled/totalled** (the bust-the-cop / disable-cruiser mechanic,
[Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) with a coarser model than the player's — 36 methods vs. the
racer's 98, because a cop needs to be *disabled*, not finely crumpled.

---

### Key takeaways

- Damage is a **mechanic family** — `DamageVehicle` base (**127 methods**, the most detailed), `DamageRacer` (98,
  player), `DamageCopCar` (36, cop), `DamageHeli` (65) — verified vtables.
- A car has **two zone systems**: coarse `DAMAGE0_*` (six regions accumulating impact) and part-specific
  `DAMAGE_*` breakables (hood, doors, bumpers, lights, glass).
- Damage shows as **deformation** — crumpled panels, broken glass (`WINDSHIELD`, `*_HEADLIGHT_GLASS`), detached
  parts (`BREAK_HEADLIGHT_*`).
- **`DamageScaleRecord`** (×24) scales a contact into zone damage; enough damage causes **performance loss** (a
  wrecked car drives worse) — separate from the reaction record.
- **Cop damage** (`DamageCopCar`, `DAMAGE_COP_LIGHTS`/`_SPOILER`) is coarser — cops get **disabled**, not finely
  crumpled.

**Next:** [Chapter 46 — AI Architecture: Goals & Actions](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md): the
minds driving the cars.
