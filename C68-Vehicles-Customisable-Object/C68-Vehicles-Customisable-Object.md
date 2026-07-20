# Chapter 68 — Vehicles: the Customisable Object

> **Goal of this chapter:** decode the car as a *customisable object* — the runtime `PlayerCar`/`CarType` model, the
> `Customize*` category screens that organise the shop (Performance, Paint, Rims, Spoiler, Decals, Numbers, HUD
> colour), the `PART_*` catalog of parts, and what pressing "buy" in the `CustomizeShoppingCart` actually does to
> the car's state.

Chapters 13 ([car-tuning vault](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and 56
([customization](../C56-Customization/C56-Customization.md)) decoded the *vault data* a car's parts edit. This
chapter decodes the *object and the flow* around that data: what a car **is** as a runtime object, how the shop is
**organised** into categories, how a **part** is named and cataloged, and what **buying** one does. It opens the
"cars" cluster ([Chapters 68–71](../C71-Cars-End-To-End/C71-Cars-End-To-End.md)) — the object here, the upgrade
mechanics in [Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md), the visual layer
in [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md), and the end-to-end walkthrough in
[Chapter 71](../C71-Cars-End-To-End/C71-Cars-End-To-End.md).

> **Verified against the executable.** The car object and shop are named in `speed.exe`: **`PlayerCar`**,
> **`CarType`**, **`CarSlot`**, **`CarPart`**, **`CustomizePart`**. The customization front-end is a family of
> `Customize*` screens — **`CustomizeMain`**/`CustomizeMainOption`/`CustomizeSub`, **`CustomizeCategory`**,
> **`CustomizeParts`**/`CustomizePartOption`, **`CustomizePerformance`**, **`CustomizePaint`**/`CustomizePaintDatum`,
> **`CustomizeRims`**, **`CustomizeSpoiler`**, **`CustomizeDecals`**, **`CustomizeNumbers`**, **`CustomizeHUDColor`**,
> and the **`CustomizeShoppingCart`** / `FEShoppingCartItem` / `ShoppingCart_BACKROOM` purchase flow. The part
> catalog is the **`PART_*`** string family, grouped by two-letter family (`PART_EN_` engine ×10, `PART_SU_`
> suspension ×7, `PART_EC_` fuel/ECU ×7, `PART_BR_` brakes ×7, `PART_WT_` ×6, `PART_TR_` transmission ×6,
> `PART_TU_` turbo ×3, `PART_TI_` tyres ×3, `PART_NO_` nitrous ×3).

---

## Deep-dive pages

- [C68.1 — The car as an object](01-car-object.md): `PlayerCar`/`CarType` — what a car *is* at runtime, and how it
  holds its parts.
- [C68.2 — The shop's categories](02-shop-categories.md): the `Customize*` category screens and the performance /
  visual split.
- [C68.3 — Parts as catalog entries](03-part-catalog.md): the `PART_XX_` family taxonomy and how a part is named
  and identified.
- [C68.4 — What "buying" does](04-buying.md): the `CustomizeShoppingCart` flow — select → cart → purchase → owned
  state + vault write + cash.
- [C68.5 — Reading vehicles in RE](05-reading-vehicles.md): navigating the car object and shop from the strings.

---

## 68.1 The car as an object

A car is a **runtime object** ([C68.1](01-car-object.md)) — the engine names it `PlayerCar` / `CarType`, built on
the class system ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) and simulated as a vehicle
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)). The *customisable* part is that the object
holds **slots** (`CarSlot`) filled by **parts** (`CarPart`), and its behaviour is read from the vault
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) keyed by which parts are installed. So "customising a
car" is *changing which parts fill the object's slots* — and the sim reads the result.

## 68.2 The shop's categories

The customization front-end is organised into **categories** ([C68.2](02-shop-categories.md)) — a `CustomizeMain`
screen branching to `CustomizePerformance`, `CustomizePaint`, `CustomizeRims`, `CustomizeSpoiler`,
`CustomizeDecals`, `CustomizeNumbers`, and `CustomizeHUDColor`. These split into the same two halves the whole
customization system uses ([C56.1](../C56-Customization/01-two-customizations.md)): **performance** (functional) and
**visual** (cosmetic — paint, rims, spoiler, decals, numbers, HUD colour). Each category is a screen that lists the
parts you can fit in that slot.

## 68.3 Parts as catalog entries

Every part is a **catalog entry** ([C68.3](03-part-catalog.md)) named `PART_<FAMILY>_<DESCRIPTION>` — e.g.
`PART_EN_COLD_AIR_INTAKE_SYSTEM`, `PART_BR_CROSS_DRILLED_ROTORS`. The two-letter family groups the catalog:
`EN` engine, `SU` suspension, `EC` fuel/ECU, `BR` brakes, `TR` transmission, `TU` turbo, `TI` tyres, `NO` nitrous,
`WT` (a sixth performance family). The part name is a **localization key** ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md))
*and* an identifier the shop uses to catalog and price the part; installing it maps to a vault effect
([Chapter 56](../C56-Customization/C56-Customization.md)).

## 68.4 What "buying" does

Buying a part runs through the **shopping cart** ([C68.4](04-buying.md)): the `CustomizeShoppingCart` collects
`FEShoppingCartItem`s (the parts you've selected), and confirming the purchase (a) debits the player's cash
(the career economy, [Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)), (b) marks the parts
**owned**, and (c) installs them into the car's slots so the vault-driven sim
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) picks them up. "Buying" is thus a *state transition* on
the car object plus a cash debit — not a file operation.

---

### Key takeaways

- A car is a **runtime object** (`PlayerCar`/`CarType`) with **slots** (`CarSlot`) filled by **parts** (`CarPart`) —
  customising means changing which parts fill the slots, and the sim reads the result from the vault
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
- The shop is organised into **`Customize*` category screens** — Performance, Paint, Rims, Spoiler, Decals, Numbers,
  HUD colour — splitting into **performance** (functional) and **visual** (cosmetic).
- Parts are **catalog entries** named `PART_<FAMILY>_<DESC>`, grouped by two-letter family (`EN`/`SU`/`EC`/`BR`/`TR`/
  `TU`/`TI`/`NO`/`WT`) — the name is both a localization key and the shop's identifier.
- **Buying** runs the `CustomizeShoppingCart` flow: debit cash, mark parts owned, install into slots — a *state
  transition*, not a file write.
- This chapter is the **object + flow**; the vault data the parts edit is [Chapter 56](../C56-Customization/C56-Customization.md)
  / [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md), the upgrade *mechanics* are
  [Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md).

**Next:** [C68.1 — The car as an object](01-car-object.md).
