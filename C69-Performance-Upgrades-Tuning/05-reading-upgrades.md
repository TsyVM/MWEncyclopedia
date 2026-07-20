# C69.5 — Reading Upgrades in RE

> **The one-sentence version:** navigate performance customization by the classes/tiers (`PerformanceLevel`,
> `PART_TU_STAGE_*`), the ratings (`PERF_*`), and the bars (`TOPSPEED`/`ACCELERATION`/`HANDLING`) — reading upgrades
> as a three-tier summary (part → class rating → car bar) over one field set the sim also reads.

[← C69.4 — From upgrade to bar to behaviour](04-upgrade-to-behaviour.md) · [Chapter 69 hub](C69-Performance-Upgrades-Tuning.md) ·
[Next: Chapter 70 — Visual Customisation →](../C70-Visual-Customisation/C70-Visual-Customisation.md)

---

## Anchors for upgrade RE

The performance-upgrade mechanics are anchored on verified strings:

- **Classes & tiers** — the nine `PART_*` families ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)),
  `PerformanceLevel`, `PART_TU_STAGE_1/2/3_TURBO_KIT` ([C69.1](01-classes-tiers.md)).
- **Ratings** — `PERF_*` (×60): `PERF_ENGINE`/`PERF_BRAKES`/`PERF_NITROUS` + `PERF_PART_<FAMILY>_<DESC>`
  ([C69.2](02-perf-ratings.md)).
- **Bars** — `TOPSPEED`, `ACCELERATION`, `HANDLING` ([C69.3](03-tuning-bars.md)).

From these, the whole upgrade system is navigable: what you upgrade (class), how far (tier), how much it scores
(rating), and what it shows (bar).

## The RE workflow

Reading the upgrade mechanics:

1. **Enumerate the classes & tiers** — `PART_*` families + `PerformanceLevel`/`STAGE_*` ([C69.1](01-classes-tiers.md)).
2. **Map the ratings** — `PERF_PART_*` → `PERF_<class>` ([C69.2](02-perf-ratings.md)); the scoring.
3. **Read the bars** — `TOPSPEED`/`ACCELERATION`/`HANDLING` ([C69.3](03-tuning-bars.md)); the car-level summary.
4. **Trace to the sim** — the fields the bars summarise are the fields the sim reads
   ([C69.4](04-upgrade-to-behaviour.md)).

The output is the full upgrade picture: three tiers of summary (part → class → car) over one field set, previewed by
the bars and realised by the sim.

## Three tiers of summary

The chapter's structure *is* a summary hierarchy — worth holding as the mental model:

- **Part** — one `PART_*` with a `PERF_PART_*` score ([C69.2](02-perf-ratings.md)).
- **Class** — a `PERF_<class>` rating rolling up its parts ([C69.1](01-classes-tiers.md)).
- **Car** — the three bars rolling up all classes ([C69.3](03-tuning-bars.md)).

Each level summarises the one below, and the *bottom* of the stack — the tuning fields
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — is what the sim actually reads
([C69.4](04-upgrade-to-behaviour.md)). So "reading upgrades" is reading this stack top-down (bar → class → part) or
bottom-up (field → part → class → bar), and knowing that the sim reads the very bottom. The elegance is that *every*
level is a view of the same underlying fields — the player sees bars, the shop shows class ratings, the sim reads
fields, and they never disagree.

## Mechanics vs data vs object

This chapter completes a three-way division of the cars material:

- **The object** ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)) — what a
  car *is* (slots, parts, buying).
- **The mechanics** (this chapter) — how upgrades *score and summarise* (classes, tiers, ratings, bars).
- **The data** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — the tuning *fields* the sim reads.

Reading performance customization completely means all three: the object holds the parts
([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)), the mechanics score them
(this chapter), and the data feeds the sim ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). The visual
half of customization — where the same object/shop machinery drives *appearance* instead of performance — is
[Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md).

## RE implications

- **Anchor on** the class/tier strings, the `PERF_*` ratings, and the three bars.
- **The RE workflow** — classes/tiers → ratings → bars → sim.
- **Three tiers of summary** — part → class → car — all views of the same fields.
- **Mechanics vs data vs object** — this chapter (scoring) vs [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)
  (fields) vs [Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md) (object).

---

### Key takeaways

- Performance upgrades are anchored on the **class/tier** strings (`PART_*` families, `PerformanceLevel`,
  `PART_TU_STAGE_*`), the **`PERF_*`** ratings, and the three **bars** (`TOPSPEED`/`ACCELERATION`/`HANDLING`).
- The RE workflow: **classes/tiers → ratings → bars → sim** — yielding a three-tier summary (part → class → car) over
  one field set.
- The system is a **summary hierarchy** — each level (part score → class rating → car bar) views the one below, and
  the **bottom** (the tuning fields) is what the **sim reads** ([C69.4](04-upgrade-to-behaviour.md)) — so no level
  can disagree.
- Reading cars-performance completely takes **three chapters**: the **object**
  ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)), the **mechanics** (this
  one), and the **data** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); the visual half is
  [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md).

**Next:** [Chapter 70 — Visual Customisation](../C70-Visual-Customisation/C70-Visual-Customisation.md).

**Sources:** `speed.exe` (verified strings: `TOPSPEED`, `ACCELERATION`/`ACCEL`, `HANDLING`; the `PERF_*` family ×60 —
`PERF_ENGINE`/`PERF_BRAKES`/`PERF_NITROUS` and `PERF_PART_<FAMILY>_<DESC>`; `PerformanceLevel`;
`PART_TU_STAGE_1/2/3_TURBO_KIT`; the nine `PART_*` classes). Bars-as-summary: [C13.5](../C13-Vault-CarTuning/05-performance-bars.md).
Sim consumers: [Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md). Catalog & object:
[Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md).
