# C68.3 — Parts as Catalog Entries

> **The one-sentence version:** every part is a catalog entry named `PART_<FAMILY>_<DESCRIPTION>`, grouped by a
> two-letter family — nine performance families (`EN` engine, `SU` suspension, `EC` fuel/ECU, `BR` brakes, `WT`
> weight, `TR` transmission, `TU` turbo, `TI` tyres, `NO` nitrous) — and the name is both a localization key and the
> shop's identifier for the part.

[← C68.2 — The shop's categories](02-shop-categories.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md) ·
[Next: C68.4 — What "buying" does →](04-buying.md)

---

## The naming scheme

Parts are cataloged by a rigid naming scheme: **`PART_<FAMILY>_<DESCRIPTION>`**. The two-letter `<FAMILY>` groups
the catalog; the `<DESCRIPTION>` names the specific part. A few verified examples:

```
PART_EN_COLD_AIR_INTAKE_SYSTEM      PART_BR_CROSS_DRILLED_ROTORS
PART_EC_PERFORMANCE_CHIP            PART_SU_COIL_OVER_SUSPENSION_SYSTEM
PART_TR_LIMITED_SLIP_DIFFERENTIAL   PART_NO_WET_SHOT_OF_NITROUS
PART_TI_EXTREME_PERFORMANCE_TIRES   PART_WT_LIGHTWEIGHT_DOORS
```

The scheme is doing double duty: the string is a **localization key** ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md))
— it looks up the display name shown in the shop — *and* an **identifier** the game uses to catalog, price, and
install the part. One string, two roles, which is why the catalog is legible from the executable: the parts *name
themselves* ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).

> ✅ *Verified:* the `PART_*` family is present in `speed.exe`, grouped as `PART_EN_` (×10), `PART_SU_` (×7),
> `PART_EC_` (×7), `PART_BR_` (×7), `PART_WT_` (×6), `PART_TR_` (×6), `PART_TU_` (×3), `PART_TI_` (×3), `PART_NO_`
> (×3) — with the descriptive suffixes above read directly from the strings.

## The nine performance families

The two-letter families *are* the performance upgrade taxonomy ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)):

| Family | Meaning | Parts include |
|---|---|---|
| `EN` | **Engine** | cold-air intake, cat-back exhaust, high-flow headers, intake manifold, downpipe, camshaft, port-and-polish heads, blueprint the block |
| `EC` | **Fuel / ECU** | engine-management unit, performance chip, fuel injectors, fuel rail, pressure regulator, high-flow pump, fuel filter |
| `TU` | **Turbo / induction** | staged forced-induction (`PART_TU_STAGE_*`) |
| `NO` | **Nitrous** | direct-port, wet shot, dry shot |
| `TR` | **Transmission** | limited-slip / differential, high-performance clutch, light flywheel, short-throw shift kit |
| `SU` | **Suspension** | coil-over system, sway bars, tie bars, performance springs & shocks |
| `BR` | **Brakes** | cross-drilled (& slotted) rotors, large-diameter rotors, race/street compound pads, steel-braided lines |
| `TI` | **Tyres** | street / pro / extreme performance tyres |
| `WT` | **Weight reduction** | lightweight doors/seats/windows, foam-filled interior, remove interior panels / rear seats |

These nine families cover the full drivetrain-and-chassis story: make power (`EN`/`EC`/`TU`/`NO`), put it down
(`TR`/`TI`), and control the car (`SU`/`BR`/`WT`). Each family feeds a specific part of the vehicle sim
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) — engine parts raise torque,
tyre and suspension parts raise grip, brake parts raise stopping force, weight parts lower mass — decoded as the
tuning bars in [Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md).

## A part maps to a vault effect

Installing a part is not cosmetic bookkeeping — it *selects vault values* the sim reads
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). The part's identifier keys into the car's tuning data:
a `PART_EN_*` engine part raises the torque curve the sim samples ([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)),
a `PART_TI_*` tyre raises the grip coefficient ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)), a
`PART_WT_*` weight part lowers the mass in the rigid body ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)).
So the catalog is the *player-facing key* into the tuning vault:

```
PART_TI_EXTREME_PERFORMANCE_TIRES  ->  install in tyre slot  ->  vault grip value  ->  sim grip
```

This is the whole point of the catalog being an identifier as well as a name ([above](#the-naming-scheme)): the same
`PART_*` string that labels the shop button is the key that, once the part is owned and installed
([C68.4](04-buying.md)), points the sim at the upgraded value. The visual parts
([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)) work the same way but key the *renderer*
(a mesh/texture) instead of the sim.

## RE implications

- **`PART_<FAMILY>_<DESC>`** — the part naming scheme; the string is both a localization key and a catalog
  identifier.
- **Nine performance families** — `EN`/`EC`/`TU`/`NO` (power), `TR`/`TI` (transfer), `SU`/`BR`/`WT` (control) — the
  verified upgrade taxonomy.
- **A part maps to a vault effect** — the identifier keys the car's tuning data
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); installing selects the upgraded value the sim reads.
- **Visual parts** key the renderer instead ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)).

---

### Key takeaways

- Parts are **catalog entries** named `PART_<FAMILY>_<DESCRIPTION>` — the string is a **localization key** *and* the
  shop's **identifier** for the part (parts name themselves, so the catalog is legible from the exe).
- The **nine performance families** are verified: `EN` engine, `EC` fuel/ECU, `TU` turbo, `NO` nitrous, `TR`
  transmission, `SU` suspension, `BR` brakes, `TI` tyres, `WT` weight — the full **make-power / put-it-down /
  control** story.
- **A part maps to a vault effect** — its identifier keys the car's tuning data
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); installing selects the upgraded value the sim reads
  every tick ([C68.1](01-car-object.md)).
- Each family feeds a specific sim system ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md))
  — torque, grip, braking, mass — surfaced as the tuning **bars**
  ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)).
- Verified part counts: `EN`×10, `SU`×7, `EC`×7, `BR`×7, `WT`×6, `TR`×6, `TU`×3, `TI`×3, `NO`×3.

**Continue:** [C68.4 — What "buying" does](04-buying.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md)
