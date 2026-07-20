# Chapter 13 — Vault Categories: Car Tuning

> **Goal of this chapter:** apply the reflection model to the data that defines how every car drives — the
> physics-behavior collections, the per-car performance values, and the tuning knobs behind the game's
> performance bars — so you can read a car's real numbers and retune them.

Chapters 11 and 12 gave you the vault as a system: typed records, reflection hashes, and `default`
inheritance. This chapter is the first of two content tours, and it takes the most-modded category of all:
**car tuning**. Everything about how a car accelerates, grips, shifts, and tops out is attribute data —
resolved through the same schema you now know — and this chapter maps where it lives and how a stored value
becomes on-road behaviour.

> **Verified against retail data.** The tuning collections are located in the live `GLOBAL/attributes.bin` by
> reflection hash: `EngineRacer = 0xB2809518`, `SuspensionRacer = 0x6209E06A`, `Physics = 0x09900113`,
> `car = 0xA13753EB` — each a real record inheriting from `default` (`0xEEC2271A`) and carrying `Float`-typed
> fields. The player-car behavior family (`EngineRacer`, `SuspensionRacer`, `DamageRacer`, `SoundRacer`) and
> the AI/cop vehicle roster (`COPMIDSIZE`, `COPSPORT`, `COPSUV`, `COPHELI`, …) are confirmed in the string
> table.

---

## Deep-dive pages

- [C13.1 — The car-tuning collection map](01-collection-map.md): the physics behaviors, the `car`/`Physics`
  collections, and the AI/cop vehicle roster — with verified hashes.
- [C13.2 — Physics behavior classes](02-behavior-classes.md): `EngineRacer`, `SuspensionRacer`, `DamageRacer`
  — how MW models a car as a set of swappable behaviors.
- [C13.3 — Reading a car's performance](03-reading-performance.md): resolve-then-decode applied to tuning
  fields, and how to label them.
- [C13.4 — Tuning value → simulation knob](04-value-to-sim.md): how a stored `Float` becomes acceleration,
  grip, or top speed in the driving model.
- [C13.5 — The performance bars](05-performance-bars.md): the top-speed / acceleration / handling UI and what
  it summarises.
- [C13.6 — Retuning a car safely](06-retuning.md): overriding per-car, editing behaviors, and keeping the
  vault consistent.

---

## 13.1 Cars are behaviors, not one struct

Most Wanted does not store a car as a single "car stats" blob. It composes a car from **behavior
collections** — an engine model, a suspension model, a damage model, a sound model — each a vault collection
with its own tuning fields. The player-car family is the **`…Racer`** set: `EngineRacer` (0xB2809518),
`SuspensionRacer` (0x6209E06A), `DamageRacer`, `SoundRacer`. AI and police cars use parallel families
(`…Traffic`, `…Cop`), and the roster of police vehicles — `COPMIDSIZE`, `COPSPORT`, `COPSUV`, `COPHELI` and
dozens more — are their own collections ([C13.1](01-collection-map.md)). A specific car selects and overrides
these behaviors.

## 13.2 Swappable models

The behavior split is deliberate: it lets the simulation swap the *engine* model independently of the
*suspension* model, and lets AI, traffic, and player cars reuse the same components with different tuning.
`EngineRacer` defines the powertrain response; `SuspensionRacer` the grip and body dynamics; `DamageRacer`
how collisions degrade the car. Each is a collection inheriting from `default`
([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)) and overriding the fields that make *this*
behavior distinct ([C13.2](02-behavior-classes.md)).

## 13.3 Reading real numbers

Reading a car's performance is the resolve-then-decode operation of
[Chapter 12](../C12-Reflection-Schema/05-resolving-values.md) applied to tuning fields: hash the collection
and field names, find the record, resolve the value through the parent chain, and decode it as its type —
overwhelmingly `Float` for tuning ([C13.3](03-reading-performance.md)). The verified `EngineRacer` record, for
instance, resolves to a set of `Float` fields whose values are the engine's tuning constants.

## 13.4 From number to behaviour

A stored value is only meaningful through the simulation that consumes it. A power figure scales
acceleration; a grip figure scales cornering force; a top-speed/gearing figure caps velocity. The chapter
maps the categories of tuning knob to the driving behaviour they produce, so you can predict what a change
*feels* like before you test it ([C13.4](04-value-to-sim.md)) — and understand why two cars with the same
top-speed number can accelerate very differently.

## 13.5 The bars are a summary

The garage's **performance bars** — top speed, acceleration, handling — are a *summary* of the underlying
tuning, not the tuning itself. They compress many `Float` fields into three human-legible ratings, which is
why upgrading a part nudges a bar: the part changes the underlying values, and the bar re-summarises
([C13.5](05-performance-bars.md)). Editing the bars means editing the values behind them.

---

### Key takeaways

- MW composes a car from **behavior collections** (`EngineRacer`, `SuspensionRacer`, `DamageRacer`,
  `SoundRacer`), not one stat block — all verified in the live vault.
- AI/traffic/cop cars use parallel behavior families and their own roster collections (`COPSPORT`, `COPSUV`,
  …).
- Tuning fields are `Float`, resolved through `default` inheritance and read by resolve-then-decode.
- A stored value becomes behaviour only through the simulation that consumes it (power→accel, grip→cornering,
  gearing→top speed).
- The garage performance bars summarise the underlying tuning; edit the values to move the bars.

**Next:** [Chapter 14 — Vault Categories: Pursuit, Surfaces & Gameplay](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md):
the police, the world's surfaces, and the gameplay rules.
