# C45.6 — Reading Damage in RE

> **The one-sentence version:** navigate damage by the `Damage*` vtables (`DamageVehicle` `0x008AD288`/127, …), the
> zone strings (`DAMAGE0_*`, `DAMAGE_*`), and `DamageScaleRecord` (×24) — reading the damage system as classes
> (behaviour) + zones (structure) + records (tuning).

[← C45.5 — Cop damage & the bust](05-cop-damage.md) · [Chapter 45 hub](C45-Damage-Deformation.md) ·
[Next: Chapter 46 — AI Architecture: Goals & Actions →](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)

---

## Anchors for damage RE

The damage system is anchored three ways:

- **The `Damage*` vtables** — `DamageVehicle` (`0x008AD288`/127), `DamageRacer` (`0x008AD2FC`/98), `DamageCopCar`
  (`0x008AD3F4`/36), `DamageHeli` (`0x008AD380`/65) ([C45.1](01-damage-family.md)).
- **The zone strings** — coarse `DAMAGE0_*` and part-specific `DAMAGE_*` ([C45.2](02-damage-zones.md)) — grep the
  exe to recover the whole zone/part vocabulary.
- **`DamageScaleRecord`** (`0xD99B853C`, ×24) and `DamageParams` (`0x7BA51D5C`) — the scaling and schema
  ([C45.4](04-scaling-performance.md)).

From these, the damage system is navigable: the classes (per-role behaviour), the zones (car structure), and the
records (tuning).

## The RE workflow

Reading damage:

1. **Enumerate the classes** — the `Damage*` vtables ([C45.1](01-damage-family.md)); the method counts show the
   per-role fidelity (127 base, 98 racer, 36 cop).
2. **Recover the zones** — grep `DAMAGE0_*` and `DAMAGE_*` ([C45.2](02-damage-zones.md)); the strings are the
   car's damage structure (coarse zones + breakables).
3. **Find the tuning** — `DamageScaleRecord` (force→damage, [C45.4](04-scaling-performance.md)) and the
   `DamageParams` fields ([C45.1](01-damage-family.md)) in the vault.
4. **Trace the feedback** — how damage degrades engine/suspension/tyres
   ([C42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) — the performance loop.

The output is the full damage picture: behaviour, structure, tuning, and feedback.

## Grep the zones: structure from strings

The cleanest way into the damage system is to **grep the zone strings** — they lay out the car's damage structure
directly ([C45.2](02-damage-zones.md)):

```bash
# the two zone systems, straight from the executable
rg -a -o 'DAMAGE0_[A-Z]+'  speed.exe   # coarse: FRONT, FRONTLEFT, ... (6)
rg -a -o 'DAMAGE_[A-Z_]+'  speed.exe   # parts:  HOOD, LEFT_DOOR, LEFT_HEADLIGHT, COP_LIGHTS, ...
```

The results *are* the damage model's structure — six coarse zones and the full breakable-part list, including
cop-specific parts ([C45.5](05-cop-damage.md)). This is a case where the string table is the most direct
documentation: the engine names every zone and part, so recovering the damage structure is a grep. No inference
needed — the car's damageable anatomy is written in the executable
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)).

## Damage completes the consequence layer

With damage decoded, the **consequence** of touching the world
([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) is complete. The discrete side of the world
membrane now has its full fan-out ([C43.3](../C43-Collision-Contacts/03-classification.md)):

- **Reaction** ([C43.4](../C43-Collision-Contacts/04-reactions.md)) — how the car moves.
- **Damage** (this chapter) — how the car is hurt (zones, deformation, performance loss).
- **Presentation** — the sparks and sound.

And damage's **feedback** ([C45.4](04-scaling-performance.md)) closes a loop back into the sim — the one output
mechanic ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) that changes the physics. So the vehicle
chapters ([39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)–45) now cover the whole car: it simulates
([39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), is built of mechanics
([40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) on a rigid body
([41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)), drives on tyres
([42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)), touches the world
([43](../C43-Collision-Contacts/C43-Collision-Contacts.md)–44), and wears the damage (this chapter). What remains
is the *mind* driving it — the AI ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)+).

## RE implications

- **Anchor on** the `Damage*` vtables, the `DAMAGE0_*`/`DAMAGE_*` zone strings, and `DamageScaleRecord`/`DamageParams`.
- **The RE workflow** — classes (fidelity) → zones (structure) → tuning (records) → feedback (performance).
- **Grep the zones** — the string table *is* the car's damage structure, recovered directly.
- **Damage completes the consequence layer** — reaction + damage + presentation, with damage's feedback into the
  sim.

---

### Key takeaways

- The damage system is anchored on the **`Damage*` vtables** (per-role fidelity), the **zone strings** (`DAMAGE0_*`
  + `DAMAGE_*`, the car's damage structure), and **`DamageScaleRecord`**/`DamageParams` (tuning).
- The RE workflow: **classes → zones → tuning → feedback** — behaviour, structure, tuning, and the performance
  loop.
- **Grepping the zone strings** recovers the damage structure directly — the executable names every zone and part
  (including cop-specific ones).
- **Damage completes the consequence layer** — reaction (move) + damage (hurt) + presentation — and its
  **feedback** ([C45.4](04-scaling-performance.md)) is the one output mechanic that loops back into the sim.
- The vehicle chapters (39–45) now cover the **whole car**; what remains is the **mind** driving it — the AI
  ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)+).

**Next:** [Chapter 46 — AI Architecture: Goals & Actions](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md): the
minds driving the cars.

**Sources:** `speed.exe` (verified: `Damage*` vtables/method counts — `DamageVehicle` `0x008AD288`/127,
`DamageRacer` `0x008AD2FC`/98, `DamageCopCar` `0x008AD3F4`/36, `DamageHeli` `0x008AD380`/65; zone strings `DAMAGE0_*`
and `DAMAGE_*` incl. `DAMAGE_COP_LIGHTS`/`_SPOILER`, glass `WINDSHIELD`/`*_HEADLIGHT_GLASS`); `GLOBAL/attributes.bin`
(verified: `DamageScaleRecord` `0xD99B853C` ×24, `DamageVehicle` ×1, `DamageRacer` ×3 as reflection-hash keys).
