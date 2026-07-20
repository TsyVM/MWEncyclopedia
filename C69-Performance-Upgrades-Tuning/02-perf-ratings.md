# C69.2 — The `PERF_` Rating System

> **The one-sentence version:** every part carries a `PERF_` entry — `PERF_PART_<FAMILY>_<DESC>` per part, rolled
> into per-category ratings `PERF_ENGINE` / `PERF_BRAKES` / `PERF_NITROUS` — the performance-scoring vocabulary that
> lets the shop tell you what an upgrade adds before you buy it.

[← C69.1 — The upgrade classes & tiers](01-classes-tiers.md) · [Chapter 69 hub](C69-Performance-Upgrades-Tuning.md) ·
[Next: C69.3 — The three tuning bars →](03-tuning-bars.md)

---

## A `PERF_` entry per part

Parallel to the `PART_*` catalog ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)) runs a second,
larger family: **`PERF_*`**, ×60 in the executable. Its structure mirrors the catalog exactly:

```
PERF_PART_EN_COLD_AIR_INTAKE_SYSTEM     PERF_PART_BR_RACE_COMPOUND_BRAKE_PADS
PERF_PART_EC_PERFORMANCE_CHIP           PERF_PART_NO_WET_SHOT_OF_NITROUS
```

Every `PART_*` part has a matching `PERF_PART_*` entry. Where the `PART_*` string *names* the part (its shop label,
[C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)), the `PERF_PART_*` string carries its
**performance dimension** — what the part *does to the car's rating*. So the two families are a name/score pair: one
labels the button, the other scores the effect.

> ✅ *Verified:* the `PERF_*` family is present in `speed.exe` (×60), structured as `PERF_PART_<FAMILY>_<DESC>` with
> one entry per `PART_*` part, alongside the category ratings below.

## The category ratings

Above the per-part entries sit **per-category ratings**:

```
PERF_ENGINE     PERF_BRAKES     PERF_NITROUS
```

These are the *rolled-up* score for a whole class ([C69.1](01-classes-tiers.md)) — the engine's total performance,
the brakes' total, the nitrous. Installing a higher-tier part in a class raises that class's category rating, which
is what the shop shows: not just "this part exists" but "your engine goes from *here* to *there*." The category
rating is the sum the player actually reasons about — you upgrade *the engine*, not *this specific intake* — and the
`PERF_ENGINE`/`PERF_BRAKES`/`PERF_NITROUS` entries are those class totals.

> 🟡 *Reasoned:* that `PERF_ENGINE`/`PERF_BRAKES`/`PERF_NITROUS` are per-*category* rollups of the per-part
> `PERF_PART_*` contributions is the natural reading of the two-level `PERF_` structure and the class model
> ([C69.1](01-classes-tiers.md)); the exact rollup arithmetic is vault/UI data. The `PERF_*` strings and their
> two-level structure (category + per-part) are verified.

## Name, score, and preview

The `PART_*` / `PERF_*` pairing is what makes the shop *informative* rather than blind:

- **`PART_*`** — the name and identity ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)): *what it
  is*.
- **`PERF_*`** — the performance score: *what it does*.
- **Together** — the shop can show a part's name *and* preview its effect on the rating before you commit
  ([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)).

This is why you can shop *strategically* — compare a stage-2 turbo against a stage-3, see which moves your engine
rating more, and decide. The `PERF_` system is the game *quantifying its own catalog* so the player can make
informed upgrade choices. It's also a gift to RE ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)):
the parallel `PART_`/`PERF_` families mean the executable carries both *what each part is called* and *that it has a
scored effect* — the catalog and its scoring, side by side.

## The rating vs the bars

A `PERF_` category rating is *not* the same as a garage **bar** ([C69.3](03-tuning-bars.md)) — they're different
levels of summary:

- **`PERF_ENGINE`** — the *engine class's* rating: how good your engine build is.
- **`TOPSPEED` / `ACCELERATION` / `HANDLING`** — the *car's* bars: how the whole build performs, across all classes
  ([C69.3](03-tuning-bars.md)).

So the engine rating feeds the top-speed and acceleration bars, the tyre/suspension/brake ratings feed handling, and
so on — the class ratings are *inputs*, the bars are the *car-level outputs* ([C69.4](04-upgrade-to-behaviour.md)).
Reading the `PERF_` system correctly means seeing it as the *middle* layer: parts carry `PERF_PART_*` scores, those
roll into `PERF_<class>` ratings, and those feed the three bars — three tiers of summary from the individual part up
to the whole car.

## RE implications

- **`PERF_PART_<FAMILY>_<DESC>`** — a performance score per part, mirroring the `PART_*` catalog one-to-one.
- **`PERF_ENGINE`/`PERF_BRAKES`/`PERF_NITROUS`** — per-category rollups (the class totals the player reasons about).
- **Name/score pair** — `PART_*` labels, `PERF_*` scores; together they make the shop a *preview*.
- **Middle layer** — parts → `PERF_<class>` ratings → the three bars ([C69.3](03-tuning-bars.md)).

---

### Key takeaways

- A second family, **`PERF_*`** (×60), runs parallel to the catalog — one **`PERF_PART_<FAMILY>_<DESC>`** per part —
  carrying each part's **performance score** (what it does) to the catalog's name (what it is).
- Above the per-part entries sit **per-category ratings** — **`PERF_ENGINE`**, **`PERF_BRAKES`**, **`PERF_NITROUS`**
  — the **rolled-up** class totals the player actually reasons about when upgrading.
- The **`PART_`/`PERF_` pairing** makes the shop a **preview** — name *and* effect — so you can shop strategically
  ([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)).
- The `PERF_` ratings are the **middle layer**: individual part scores roll into **class ratings**, which feed the
  three **car-level bars** ([C69.3](03-tuning-bars.md)) — three tiers of summary.
- Verified: the `PERF_*` family (×60) and its two-level structure (category ratings + per-part contributions).

**Continue:** [C69.3 — The three tuning bars](03-tuning-bars.md) · [Chapter 69 hub](C69-Performance-Upgrades-Tuning.md)
