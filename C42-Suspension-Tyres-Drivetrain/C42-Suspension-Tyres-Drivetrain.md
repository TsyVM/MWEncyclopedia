# Chapter 42 — Suspension, Tyres & Drivetrain

> **Goal of this chapter:** decode the wheel-force half of the vehicle sim — the `Engine*` drivetrain family, the
> `Suspension*` family, and the tyre model — the mechanics that turn engine torque and wheel loads into the
> forces the rigid body integrates, all anchored on verified vtables and vault keys.

Between "the driver presses the throttle" and "the rigid body moves" sit the **drivetrain, suspension, and
tyres** — the mechanics that compute the actual forces at the four contact patches. This chapter decodes the
`Engine*` and `Suspension*` class families (the `BEHAVIOR_MECHANIC_ENGINE` and `_SUSPENSION` implementations,
[Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) and the tyre model that couples them to the road. It
is the deepest layer of the driving model — where a car's *performance* and *handling* are actually computed.

> **Verified against the executable and vault.** The `Engine*` and `Suspension*` families are byte-verified
> classes in `speed.exe`, each a real vtable of code pointers: `EngineRacer` (hash `0xB2809518`, vtable
> `0x008AB6A0`, **123 methods**), `EngineTraffic` (`0x5C216BAB`, `0x008AB8F8`, 67), `EngineDragster` (`0x4BC7F9AF`,
> `0x008ABF34`, 132), `EngineSpline` (`0x3F172A3E`, `0x008AB7AC`, 56); `SuspensionRacer` (`0x6209E06A`,
> `0x008ABAC0`, 45), `SuspensionTraffic` (`0x12D5313C`, `0x008ABB80`, 86), `SuspensionSimple` (`0x723B315B`,
> `0x008ABC28`, 44), `SuspensionSpline` (`0xBB35585B`, `0x008ABD88`, 57), `SuspensionTrailer` (`0xD44C9372`,
> `0x008ABCE0`, 99). Every method count was confirmed by counting consecutive code pointers at the vtable. The
> concrete specs are **vault keys** in `GLOBAL/attributes.bin`: `EngineRacer` ×4, `EngineTraffic` ×2,
> `SuspensionRacer` ×3, `SuspensionTraffic` ×2. Tyre-damage states `ETireBlown`/`ETirePunctured` and the
> `Drivetrain`/`POTransmission` classes are present as strings.

---

## The two families

| Family | Racer (hero) | Traffic (cheap) | Simple | Spline (rail) | Trailer |
|---|---|---|---|---|---|
| **Engine** | `EngineRacer` (123) | `EngineTraffic` (67) | — | `EngineSpline` (56) | — |
| **Suspension** | `SuspensionRacer` (45) | `SuspensionTraffic` (86) | `SuspensionSimple` (44) | `SuspensionSpline` (57) | `SuspensionTrailer` (99) |

(Method counts in parentheses — all verified.) Plus `EngineDragster` (132) for the drag minigame.

---

## Deep-dive pages

- [C42.1 — Fidelity tiers](01-fidelity-tiers.md): the Racer/Traffic/Simple/Spline/Trailer tiers and shared shell.
- [C42.2 — The engine & drivetrain](02-engine-drivetrain.md): `EngineRacer` — torque, gearbox, clutch, NOS,
  drive split.
- [C42.3 — Suspension](03-suspension.md): `SuspensionRacer` — spring/damper, ride height, anti-roll, load
  transfer.
- [C42.4 — The tyre model](04-tyres-grip.md): slip, load sensitivity, grip, and tyre-damage states.
- [C42.5 — The tuning surface](05-tuning-surface.md): gear ratios, torque, NOS, spring rates — all vault data.
- [C42.6 — Reading the drivetrain in RE](06-reading-drivetrain.md): navigating the families by vtable and vault.

---

## 42.1 Fidelity tiers

The `Engine*` and `Suspension*` families come in **fidelity tiers** ([C42.1](01-fidelity-tiers.md)) so a traffic
car and a hero car share the vehicle shell but pay different simulation costs: `*Racer` is the full model (the
player), `*Traffic` is the cheap model (ambient cars), `*Simple` and `*Spline` are intermediate/rail variants,
`*Trailer` handles towed bodies ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)). All register onto the
mechanics list-head at `0x0092C660` ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)). The tier is
chosen per car by which spec its vault data references.

## 42.2 The engine & drivetrain

`EngineRacer` (**123 methods** — the most of the engine family) is the **full drivetrain sim**
([C42.2](02-engine-drivetrain.md)): torque curve, gearbox, clutch, NOS, and the drive split to the wheels — this
is where throttle becomes wheel force, and the model behind the garage's top-speed/acceleration bars
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). `EngineTraffic` (67 methods) is the cheap
version for ambient cars; `EngineDragster` (132) adds the launch/perfect-shift mechanics of the drag minigame.

## 42.3 Suspension

`SuspensionRacer` (45 methods) is the **full suspension** ([C42.3](03-suspension.md)): per-wheel spring/damper,
ride height, anti-roll, and load transfer — the model that makes a tuned car feel planted or loose. Its
constructor references `SuspensionParams` (the base parameter class). Curiously, `SuspensionTraffic` has *more*
methods (86) than the racer (45) — because its lightweight per-wheel approximations are applied across the whole
busy traffic population and kept inline ([C42.1](01-fidelity-tiers.md)).

## 42.4 The tyre model

The **tyre model** ([C42.4](04-tyres-grip.md)) is where suspension load and drivetrain torque become the actual
longitudinal and lateral forces on the body: each wheel computes a force from its **slip** (ratio for
drive/brake, angle for cornering), scaled by its **load** (from the suspension, [C42.3](03-suspension.md)) and the
**surface grip** ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)). Tyres can be damaged — the verified
states `ETireBlown` and `ETirePunctured` are the spike-strip/wear outcomes
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).

## 42.5 The tuning surface

Everything in this chapter is **tunable by data** ([C42.5](05-tuning-surface.md)): gear ratios, torque, NOS,
spring rates, ride height, anti-roll, mass transfer — all vault fields in the per-car collections
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). The `Engine*`/`Suspension*` classes are the code
that *consumes* those numbers; a performance mod edits the numbers, not the classes. This is the heart of car
tuning — the editing surface the whole garage sits on.

---

### Key takeaways

- The drivetrain and suspension are two **verified class families** — `Engine*` (Racer 123, Traffic 67, Dragster
  132, Spline 56) and `Suspension*` (Racer 45, Traffic 86, Simple 44, Spline 57, Trailer 99).
- They come in **fidelity tiers** — hero (`*Racer`), traffic (cheap), simple, rail (`*Spline`), trailer — sharing
  the vehicle shell at different costs, all on list-head `0x0092C660`.
- **`EngineRacer`** turns throttle into wheel force (torque curve, gearbox, clutch, NOS); **`SuspensionRacer`**
  computes wheel loads and weight transfer.
- The **tyre model** turns load + torque + slip + surface grip into the forces the rigid body integrates; tyres
  can be blown/punctured.
- Everything is **vault-tuned** — gear ratios, spring rates, NOS — the classes consume the numbers; the tuning is
  data.

**Next:** [Chapter 43 — Collision Detection & Contact Records](../C43-Collision-Contacts/C43-Collision-Contacts.md):
how the bodies touch the world.
