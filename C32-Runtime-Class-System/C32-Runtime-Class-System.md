# Chapter 32 — The Runtime Class System & Object Model

> **Goal of this chapter:** cross from files on disk to the *running game* — the class system that turns loaded
> data into live objects, the five roles those classes play, the eleven families they register into, and the
> object model (size + vtable + fields) that makes them behave.

Everything so far has been *data at rest* — textures, geometry, vaults, audio, saves. The running game is *data
in motion*: loaded content becomes **live objects** of C++ **classes** that update every frame. This chapter is
the pivot — the class system that binds the formats to behaviour. It's grounded in the shipped executable, and
it reuses a hash you already know.

> **Verified against the executable.** The runtime registers each class at startup onto one of **eleven global
> list-heads**, keying it by a hash of its class name — and that hash is the **same reflection hash** as the
> vault: `lookup2` seeded `0xABCDEF00`. Confirmed live: `reflection_hash("EngineRacer") = 0xB2809518` and
> `("SuspensionRacer") = 0x6209E06A` (matching the vault, [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md));
> the Jenkins constant `0x9E3779B9` and the seed `0xABCDEF00` are present in `speed.exe` (4 and 50 times), and
> the family list-heads (`0x0092C660` mechanics, `0x0090D8EC` AI actions, …) are referenced throughout the
> binary.

---

## Deep-dive pages

- [C32.1 — Data becomes live objects](01-data-to-objects.md): how loaded content is instantiated as class
  objects.
- [C32.2 — The five class roles](02-five-roles.md): the roles a runtime class plays.
- [C32.3 — The eleven families](03-eleven-families.md): the registration lists and their sizes, with verified
  list-head addresses.
- [C32.4 — Registration: name → hash → list-head](04-registration.md): the startup registration keyed by the
  reflection hash.
- [C32.5 — The object model](05-object-model.md): size + vtable + fields — what a live object is.
- [C32.6 — Reading the class system from the binary](06-reading-binary.md): identifying classes and behaviour
  in `speed.exe`.

---

## 32.1 Loaded data becomes objects

A file on disk is inert; the game makes it *do* something by instantiating **objects**. A car's tuning
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) becomes a live vehicle object; a cop becomes an
`AIVehicleCopCar`; a trigger ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)) becomes a live
trigger object. These objects are C++ class instances that hold state and update each frame
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)). So the class system is the bridge: **data
in → object out → behaviour** ([C32.1](01-data-to-objects.md)).

## 32.2 Five roles

The runtime classes fall into a handful of **roles** — the kinds of thing a class *is* in the running game
([C32.2](02-five-roles.md)): entities (cars, cops), the mechanic components that make them work, the AI that
drives them, the managers/systems that coordinate them, and the connectors that wire them together. Knowing a
class's role is most of understanding it.

## 32.3 Eleven families

More precisely, classes register into **eleven families**, each a global list the runtime walks — verified by
their list-head addresses in `speed.exe`:

| Family | List-head | Count |
|---|---|---|
| Vehicle **mechanics** | `0x0092C660` | 51 |
| **AI actions** | `0x0090D8EC` | 16 |
| **Managers & activities** | `0x0092C668` | 14 |
| **AI goals** | `0x0090D8E8` | 14 |
| **Connectors** | `0x00988EC0` | 8 |
| **Director actions** | `0x009111FC` | 7 |
| **Named systems** | `0x00988DFC` | 5 |
| **World bodies** | `0x0092C66C` | 3 |
| **Players** | `0x00988EC4` | 3 |
| **World tasks** | `0x00988EBC` | 2 |
| **Devices** | `0x00920464` | 1 |

The mechanics family (51) dominates — the vehicle simulation is the game's largest class population
([C32.3](03-eleven-families.md), [Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).

## 32.4 Registration by reflection hash

At startup each class **registers** itself: it hashes its name with the reflection hash
([C2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md), `lookup2`/`0xABCDEF00`) and links itself
onto its family's list-head ([C32.4](04-registration.md)). This is the same hash the vault uses for fields
([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)) — verified, `EngineRacer` hashes to `0xB2809518` both
as a class and as a vault collection. So the class registry, the vault, and the reflection system are **one hash
world** — a genuine unification.

## 32.5 The object model

A live object is **size + vtable + fields** ([C32.5](05-object-model.md)): a block of memory of a fixed **size**
(e.g. `AIVehicleCopCar` = 1964 bytes), beginning with a **vtable pointer** (to its virtual methods,
[Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)), followed by its **fields** (state). The size and
method count are the object's fingerprint — `AIVehicleCopCar`'s 324 vtable methods mark it as the most
behaviour-packed class in the game.

---

### Key takeaways

- The class system turns **loaded data into live objects** — the pivot from files to the running game.
- Classes play a few **roles** and register into **eleven families** (verified list-heads; mechanics = 51,
  the largest).
- Registration keys each class by the **reflection hash** (`lookup2`/`0xABCDEF00`) — the *same* hash as the
  vault (`EngineRacer` → `0xB2809518`, verified).
- The class registry, vault, and reflection system are **one hash world**.
- A live object is **size + vtable + fields**; size/method-count fingerprint a class (`AIVehicleCopCar` = 1964
  B, 324 methods).

**Next:** [Chapter 33 — The Class Registry, Factories & the Class Reference](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md):
how objects are constructed, and a catalogue of the classes.
