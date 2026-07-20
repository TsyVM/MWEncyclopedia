# Chapter 33 — The Class Registry, Factories & the Class Reference

> **Goal of this chapter:** decode how the runtime constructs objects — the intern function that hashes class
> names, the factory registration that binds a name to a constructor, and the resulting catalogue of the game's
> classes with their verified sizes and vtables.

Chapter 32 established that classes register into eleven families keyed by the reflection hash. This chapter is
the *mechanism* — the **intern function** that turns a class name into a key, the **factory** that constructs an
object by name, and the **class reference**: a family-by-family catalogue of what the game actually
instantiates, grounded in the executable.

> **Verified against the executable.** The intern function is **`0x5CC240`** — confirmed live: it reads a
> string argument (`mov eax,[esp+4]; test; cmp byte [eax],0`), then hashes it (`lookup2`/`0xABCDEF00`, wrapping
> the core hash at `0x5CC090`). A linear scan finds **686 call sites** to `0x5CC240`; ~**124** are class
> registrations, the rest general name-hashing. The largest catalogued class is `AIVehicleCopCar` (1964 bytes,
> 324 vtable methods). ImageBase `0x400000`, RVA == file-offset.

---

## Deep-dive pages

- [C33.1 — The intern function](01-intern.md): `0x5CC240` — hashing a class name to its key.
- [C33.2 — Factory registration](02-factory-registration.md): binding name → key → constructor → list-head.
- [C33.3 — Constructing an object](03-construction.md): from a name to a live instance.
- [C33.4 — The class reference](04-class-reference.md): the family-by-family catalogue.
- [C33.5 — Sizes & vtables as fingerprints](05-fingerprints.md): reading a class from its measurements.
- [C33.6 — Using the registry in RE](06-using-registry.md): navigating classes via the registry.

---

## 33.1 The intern function `0x5CC240`

Every class registration begins by pushing its name string and calling **`0x5CC240`** — the *intern* function.
Verified from its bytes, it takes a `const char*`, checks it's non-null and non-empty, and hashes it with the
reflection hash ([C2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)) — wrapping the core Jenkins
`lookup2` at `0x5CC090` ([C33.1](01-intern.md)). It returns the 32-bit key. This is the one function that turns
a class name into the key everything else uses.

A crucial caveat: a call to `0x5CC240` means "hash this name," **not** necessarily "register a class." Of its
686 call sites, only ~124 are registrations; the rest hash names for other purposes (vault fields, lookups). So
you read *what the caller does with the key* to tell a registration from a plain hash
([C33.2](02-factory-registration.md)).

## 33.2 Factory registration

A **factory registration** binds four things ([C33.2](02-factory-registration.md)): the class **name** (interned
to a key via `0x5CC240`), a **constructor** (the function that builds an instance), the **vtable** (its
behaviour), and the **family list-head** ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)) it links
onto. After registration, the runtime can construct "a class named X" by hashing X, finding its registration,
and calling its constructor.

## 33.3 Construction by name

Constructing an object is the registry's payoff ([C33.3](03-construction.md)): given a name (from the vault,
[C13.2](../C13-Vault-CarTuning/02-behavior-classes.md), or another system), hash it, find the registered
factory, allocate the object's memory ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)), and run
the constructor — producing a live instance ([C32.1](../C32-Runtime-Class-System/01-data-to-objects.md)). This
is **data-driven construction**: data names a class, the factory builds it — the mechanism behind the whole
class system.

## 33.4 The class reference

Walking the registrations family-by-family yields the **class reference** — a catalogue of what the game
instantiates ([C33.4](04-class-reference.md)): the 51 mechanics, the 14 AI goals and 16 actions, the managers,
connectors, and entities, each with its name, size, and vtable. This catalogue (archive Discovery 10) is the map
of the runtime — the classes the file formats configure and the behaviours they drive
([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).

## 33.5 Fingerprints

Each catalogued class carries a **size** and **vtable method count** — its fingerprint
([C33.5](05-fingerprints.md), [C32.5](../C32-Runtime-Class-System/05-object-model.md)). `AIVehicleCopCar`'s 1964
bytes and 324 methods mark it as the heaviest actor; a small connector's few bytes and methods mark it as a
wire. Role + size + method count is most of a class's story before you read its code.

---

### Key takeaways

- The **intern function `0x5CC240`** hashes a class name to its key (reflection hash, wrapping `0x5CC090`) —
  verified.
- A call to `0x5CC240` means "hash a name," not always "register a class" — 686 calls, ~124 registrations.
- **Factory registration** binds name → key → constructor → vtable → family list-head.
- **Construction by name** (hash → factory → allocate → construct) is data-driven instantiation — the class
  system's payoff.
- The **class reference** catalogues the runtime family-by-family, each class fingerprinted by size + vtable
  count (`AIVehicleCopCar` = 1964 B / 324).

**Next:** [Chapter 34 — VTable Anatomy & Method Roles](../C34-VTable-Anatomy/C34-VTable-Anatomy.md): reading a
class's behaviour from its vtable.
