# C33.4 — The Class Reference

> **The one-sentence version:** walking the ~124 registrations family-by-family yields the class reference — a
> catalogue of what the game instantiates, each class with its name, family (role), size, and vtable — the map
> of the runtime.

[← C33.3 — Constructing an object](03-construction.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md) ·
[Next: C33.5 — Sizes & vtables as fingerprints →](05-fingerprints.md)

---

## The catalogue

Enumerating the family lists ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)) — following each
list-head's registrations ([C33.2](02-factory-registration.md)) — produces the **class reference**: every class
the game registers, with its identity and measurements. It's the runtime's table of contents, and the archive's
Discovery 10 catalogued it. A sketch of what the families hold:

| Family | What's in it (examples) |
|---|---|
| Mechanics (51) | engine, transmission, suspension, tyres, steering, drivetrain, induction, body — and variants ([Ch 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) |
| AI goals (14) | pursuit, race, flee, patrol goals ([Ch 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) |
| AI actions (16) | the actions goals decompose into ([Ch 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) |
| Managers (14) | traffic, pursuit, population, activity managers ([Ch 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) |
| Connectors (8) | the wires between subsystems ([Ch 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) |
| World bodies (3), Players (3), Devices (1) | top-level entities and input |

The catalogue is where the file-format world and the behaviour world meet: each class is configured by data
(vault, geometry) and drives a subsystem.

## Reading a catalogue entry

Each entry records ([C33.5](05-fingerprints.md)):

- **Name** — the class's name (and its reflection-hash key, [C33.1](01-intern.md)).
- **Family** — which list-head it registered onto → its role ([C32.2](../C32-Runtime-Class-System/02-five-roles.md)).
- **Size** — the object's byte size ([C32.5](../C32-Runtime-Class-System/05-object-model.md)).
- **Vtable & method count** — its behaviour ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).

So an entry answers "what is this, what kind, how big, how much behaviour" — most of a class's story before you
read code.

## The catalogue is the map

The class reference is the single most useful artifact for understanding the runtime, because it turns the
6 MB executable into a **legible list**:

- **Find a subsystem's classes** — the mechanics family is the vehicle sim
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)); the AI families are the AI
  ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)).
- **Gauge the game's shape** — the family sizes ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md))
  show where complexity concentrates (cars, then AI).
- **Cross-reference data** — a class with a vault collection of the same name is the same entity
  ([C13.2](../C13-Vault-CarTuning/02-behavior-classes.md)).

So the later chapters ([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
are, in a sense, guided tours of this catalogue's families.

> ✅ *Verified (archive Discovery 10):* the class catalogue — names, families, sizes, vtable method counts — was
> recovered from the registrations; the largest is `AIVehicleCopCar` (1964 B, 324 methods).
> 🟡 *Reasoned:* the complete per-class field layouts are per-class RE; the catalogue's names/families/sizes/
> counts are the verified backbone.

## How the catalogue was built

The method ([C33.6](06-using-registry.md)): scan `.text` for the ~124 registration call sites
([C33.1](01-intern.md)), read each registration's name, family list-head, constructor, and vtable, and measure
each class's size and method count. The result is the catalogue. Because the intern hash is computable
([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)), even classes whose names weren't directly readable
can often be recovered by matching their key-hash against a name dictionary
([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)).

## RE implications

- **The catalogue is the runtime map** — start here to understand any subsystem.
- **Each entry = name + family + size + vtable** — the class's fingerprint ([C33.5](05-fingerprints.md)).
- **Build it from the ~124 registrations** ([C33.6](06-using-registry.md)).
- **Cross-reference the vault** for the data side ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).

---

### Key takeaways

- The **class reference** catalogues every registered class — name, family (role), size, vtable — the runtime's
  table of contents.
- Each entry answers what/what-kind/how-big/how-much-behaviour before you read code.
- It maps the game: mechanics = vehicle sim, AI families = AI, managers = coordinators.
- Built by scanning the ~124 registrations; computable hashes help recover class names.
- The catalogue is the map the later runtime/simulation/AI chapters tour, family by family.

**Continue:** [C33.5 — Sizes & vtables as fingerprints](05-fingerprints.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md)
