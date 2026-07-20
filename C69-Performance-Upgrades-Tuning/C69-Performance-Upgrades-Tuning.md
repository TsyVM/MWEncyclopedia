# Chapter 69 — Performance Upgrades & Tuning Bars

> **Goal of this chapter:** decode the *mechanics* of performance customization — the nine upgrade **classes** and
> their **tiers** (the `PART_*` families at `PerformanceLevel`, the `PART_TU_STAGE_1/2/3` turbo stages), the `PERF_*`
> **rating** vocabulary that scores each part's contribution, and the three garage **bars** (`TOPSPEED`,
> `ACCELERATION`, `HANDLING`) that summarise it all.

Chapter 68 decoded the *catalog* ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)) — what the parts
*are*. This chapter decodes what they *do*: how a part belongs to an upgrade **class**, sits at a **tier** within
it, carries a **`PERF_` rating**, and moves the **bars** the garage shows. It's the quantitative layer between the
catalog ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)) and the tuning vault
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — the *mechanics* companion to the vault's data
([C13.5](../C13-Vault-CarTuning/05-performance-bars.md)) and the customization screens
([C56.2](../C56-Customization/02-performance-parts.md)).

> **Verified against the executable.** The three garage bars are named in `speed.exe`: **`TOPSPEED`**,
> **`ACCELERATION`** (`ACCEL`), **`HANDLING`**. The performance-rating vocabulary is the **`PERF_*`** family (×60):
> the category ratings **`PERF_ENGINE`**, **`PERF_BRAKES`**, **`PERF_NITROUS`**, and a **`PERF_PART_<FAMILY>_<DESC>`**
> entry for every part. Upgrade tiers are named too: **`PerformanceLevel`**, and the turbo stages
> **`PART_TU_STAGE_1_TURBO_KIT`** / **`_STAGE_2_`** / **`_STAGE_3_`**. The nine upgrade classes are the `PART_*`
> families ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)): `EN`/`EC`/`TU`/`NO`/`TR`/`SU`/`BR`/`TI`/`WT`.

---

## Deep-dive pages

- [C69.1 — The upgrade classes & tiers](01-classes-tiers.md): the nine families as upgrade classes, and the
  `PerformanceLevel` / turbo-stage tiers within them.
- [C69.2 — The `PERF_` rating system](02-perf-ratings.md): the category ratings and per-part contributions that
  score an upgrade.
- [C69.3 — The three tuning bars](03-tuning-bars.md): `TOPSPEED` / `ACCELERATION` / `HANDLING` as summaries, not
  stored numbers.
- [C69.4 — From upgrade to bar to behaviour](04-upgrade-to-behaviour.md): the chain from installing a part to the
  bar moving and the car driving faster.
- [C69.5 — Reading upgrades in RE](05-reading-upgrades.md): navigating the upgrade mechanics from the strings.

---

## 69.1 The upgrade classes & tiers

The nine `PART_*` families ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)) are **upgrade
classes** ([C69.1](01-classes-tiers.md)) — engine, ECU, turbo, nitrous, transmission, suspension, brakes, tyres,
weight — and within a class parts sit at **tiers** (`PerformanceLevel`). Some classes are explicitly *staged*: the
turbo is `PART_TU_STAGE_1/2/3_TURBO_KIT`, a three-rung ladder. Upgrading a class means climbing its tiers.

## 69.2 The `PERF_` rating system

Every part carries a **`PERF_` rating** ([C69.2](02-perf-ratings.md)) — a `PERF_PART_<FAMILY>_<DESC>` entry scoring
its performance contribution, rolled up into per-category ratings (`PERF_ENGINE`, `PERF_BRAKES`, `PERF_NITROUS`).
This is the quantitative layer: the catalog names the part, the `PERF_` entry *scores* it, so the game can show how
much an upgrade adds before you buy ([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)).

## 69.3 The three tuning bars

The garage shows three **bars** ([C69.3](03-tuning-bars.md)) — `TOPSPEED`, `ACCELERATION`, `HANDLING` — that
*summarise* the car's current build. As decoded on the vault side ([C13.5](../C13-Vault-CarTuning/05-performance-bars.md)),
these are **not stored ratings** but *summaries* of the underlying tuning fields — which is exactly why installing a
part moves a bar: the part changes the fields, and the bar re-summarises them.

## 69.4 From upgrade to bar to behaviour

Installing an upgrade runs a chain ([C69.4](04-upgrade-to-behaviour.md)): the part changes the car's tuning values
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), the bar re-summarises them
([C69.3](03-tuning-bars.md)), *and* the sim ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md))
reads the new values so the car actually drives faster. The bar is the *preview*; the sim is the *reality* — both
driven by the same upgraded fields.

---

### Key takeaways

- The nine `PART_*` families are **upgrade classes**; within a class, parts sit at **tiers** (`PerformanceLevel`),
  some explicitly **staged** (turbo `STAGE_1/2/3`) — upgrading means climbing the tiers.
- Every part carries a **`PERF_` rating** (`PERF_PART_<FAMILY>_<DESC>`, rolled into `PERF_ENGINE`/`PERF_BRAKES`/
  `PERF_NITROUS`) — the score that lets the shop show what an upgrade adds.
- The garage's three **bars** — `TOPSPEED`, `ACCELERATION`, `HANDLING` — are **summaries, not stored numbers**
  ([C13.5](../C13-Vault-CarTuning/05-performance-bars.md)); upgrading moves a bar because the part changes the fields
  it summarises.
- The full chain is **part → tuning fields → bar (preview) + sim (reality)** — the bar and the driving are the same
  upgrade, seen two ways.
- This is the **mechanics** layer — the catalog is [Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md),
  the vault data is [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md), the visual side is
  [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md).

**Next:** [C69.1 — The upgrade classes & tiers](01-classes-tiers.md).
