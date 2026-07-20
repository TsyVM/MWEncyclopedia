# C69.1 — The Upgrade Classes & Tiers

> **The one-sentence version:** the nine `PART_*` families are upgrade *classes*, and within a class parts sit at
> *tiers* (`PerformanceLevel`) — some classes an explicit ladder, like the turbo's `STAGE_1/2/3` — so upgrading a
> car means climbing each class's tiers toward its best part.

[← Chapter 69 hub](C69-Performance-Upgrades-Tuning.md) · [Next: C69.2 — The `PERF_` rating system →](02-perf-ratings.md)

---

## Classes and tiers

Performance customization has two axes: **which class** you upgrade, and **how far** within it.

- A **class** is one of the nine `PART_*` families ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md))
  — engine (`EN`), ECU (`EC`), turbo (`TU`), nitrous (`NO`), transmission (`TR`), suspension (`SU`), brakes (`BR`),
  tyres (`TI`), weight (`WT`). Each class is a distinct axis of the car's performance.
- A **tier** is *how good* the part in that class is — the game tracks this as **`PerformanceLevel`**. A
  higher-tier brake (race-compound pads) outperforms a lower one (street pads); installing it *climbs* the class.

So a fully-built car is *the top tier of every class* — the best engine part, the best tyres, the best brakes. The
build is a vector of nine tiers, one per class, and "upgrading" is raising one of them. This is why performance
customization feels like a checklist: nine classes to max out.

> ✅ *Verified:* `PerformanceLevel` is a string in `speed.exe`; the nine classes are the verified `PART_*` families
> ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)).

## The turbo ladder

The clearest tiered class is the **turbo** (`TU`), which names its rungs explicitly:

```
PART_TU_STAGE_1_TURBO_KIT
PART_TU_STAGE_2_TURBO_KIT
PART_TU_STAGE_3_TURBO_KIT
```

Three stages, each a bigger turbo than the last — a literal ladder you climb from stage 1 to stage 3. The `STAGE_N`
naming makes the tier *part of the identifier* ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)), so
the progression is legible from the strings alone: the turbo class has exactly three tiers, named in order.

Other classes tier *implicitly* through their part descriptions — brakes climb street-compound pads → race-compound
pads → cross-drilled rotors → cross-drilled-and-slotted rotors; tyres climb street → pro → extreme performance.
The `STAGE_N` turbo just makes explicit what every class does: a low-to-high progression within the family.

> ✅ *Verified:* `PART_TU_STAGE_1_TURBO_KIT`, `_STAGE_2_`, and `_STAGE_3_` are strings in `speed.exe` — the three
> turbo tiers; the tyre tiers (`STREET`/`PRO`/`EXTREME_PERFORMANCE_TIRES`) are likewise named in ascending order.

## Not every class is deep

The classes differ in *how many* tiers they carry ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)
counts):

- **Deep classes** — engine (`EN`, ten parts), and the mid-depth suspension/ECU/brakes (`SU`/`EC`/`BR`, seven each)
  and transmission/weight (`TR`/`WT`, six each) — a real progression of parts to work through.
- **Shallow classes** — turbo (`TU`, three stages), tyres (`TI`, three grades), nitrous (`NO`, three shots) — a
  short ladder.

So the *shape* of the upgrade tree is uneven: the engine is a long climb, the turbo a quick three-rung jump. This
reflects the real-world domains — an engine has many independent improvements (intake, exhaust, heads, cams…), while
a turbo is essentially "how big." Reading the class depths tells you where the game puts *granularity* (the engine)
versus *discrete jumps* (the turbo), which is also where the `PERF_` ratings ([C69.2](02-perf-ratings.md)) spread
their contributions.

## RE implications

- **Two axes** — the **class** (nine `PART_*` families) and the **tier** (`PerformanceLevel`) within it.
- **The turbo ladder** — `PART_TU_STAGE_1/2/3` names its tiers explicitly; other classes tier implicitly by part.
- **Uneven depth** — engine is deep (ten parts), turbo/tyres/nitrous shallow (three) — granularity vs discrete
  jumps.
- **A build is nine tiers** — the top tier of every class is a maxed car.

---

### Key takeaways

- Performance upgrades have **two axes**: the **class** (one of the nine `PART_*` families) and the **tier**
  (`PerformanceLevel`) within it — a build is a **vector of nine tiers**, and upgrading raises one.
- The **turbo** names its tiers explicitly — `PART_TU_STAGE_1/2/3_TURBO_KIT` — a three-rung ladder; other classes
  tier **implicitly** through ascending part descriptions (tyres: street → pro → extreme).
- Class **depth is uneven** — engine (ten parts) is a long climb, turbo/tyres/nitrous (three) a quick jump —
  mirroring the real domains and where the game places granularity.
- A **maxed car** is the top tier of **every** class; performance customization is the checklist of climbing all
  nine.
- Verified: `PerformanceLevel`, the `PART_TU_STAGE_*` turbo tiers, and the nine `PART_*` classes.

**Continue:** [C69.2 — The `PERF_` rating system](02-perf-ratings.md) · [Chapter 69 hub](C69-Performance-Upgrades-Tuning.md)
