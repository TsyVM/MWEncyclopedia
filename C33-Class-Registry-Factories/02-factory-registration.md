# C33.2 — Factory Registration

> **The one-sentence version:** a registration binds four things — the class name (interned to a key via
> `0x5CC240`), a constructor, a vtable, and a family list-head — so afterward the runtime can build "a class
> named X" by hashing X and calling its constructor.

[← C33.1 — The intern function](01-intern.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md) ·
[Next: C33.3 — Constructing an object →](03-construction.md)

---

## What a registration binds

A **factory registration** — one of the ~124 registration call sites ([C33.1](01-intern.md)) — associates a
class name with everything needed to construct and identify it:

```
register "EngineRacer":
   key         = intern("EngineRacer")        ; 0x5CC240 → 0xB2809518
   constructor = <fn that builds an EngineRacer>
   vtable      = <its virtual method table>   (Chapter 34)
   link {key, constructor, vtable, …} onto the mechanics list-head (0x0092C660)
```

So a registration records: **who** (name/key), **how to build** (constructor), **how it behaves** (vtable), and
**what kind** (family list-head, [C32.3](../C32-Runtime-Class-System/03-eleven-families.md)). After all classes
register, the eleven family lists ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)) hold complete
factory entries.

## Telling a registration from a hash

Since `0x5CC240` is called for any name-hash ([C33.1](01-intern.md)), a registration is identified by what
follows the call: the returned key is stored into a record alongside a **constructor pointer** and the record is
**linked onto a family list-head**. Concretely, in the disassembly a registration thunk:

1. Pushes the class name, calls `0x5CC240` (get the key).
2. Fills a small registration struct (key, constructor, vtable).
3. Links it onto one of the eleven list-heads
   ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)).

A plain lookup, by contrast, uses the key to *search* a list rather than *add* to one. So the family list-head
write is the registration's signature ([C33.6](06-using-registry.md)).

> ✅ *Verified:* ~124 of the 686 `0x5CC240` call sites are registrations that link onto the eleven family
> list-heads (verified addresses, [C32.3](../C32-Runtime-Class-System/03-eleven-families.md)); the class→list-
> head assignment was recovered by disassembly.
> 🟡 *Reasoned:* the exact registration-struct layout (field order of key/constructor/vtable) is per-site RE
> detail; the four bound elements and the list-head link are verified.

## The factory pattern

This is the classic **factory registry** pattern: a central registry maps names to constructors, so code can
create objects by name without knowing their concrete types:

- **Decoupling.** The creator names a class ("make an `EngineRacer`") without a compile-time dependency on the
  class — the registry resolves it ([C33.3](03-construction.md)).
- **Extensibility.** Adding a class is adding a registration; the creating code is unchanged.
- **Data-driven.** Data (the vault, [C13.2](../C13-Vault-CarTuning/02-behavior-classes.md)) can name a class,
  and the registry builds it — the mechanism behind data-driven construction.

So the factory registry is what lets the vault's `EngineRacer` collection instantiate the `EngineRacer` class:
the shared key ([C32.4](../C32-Runtime-Class-System/04-registration.md)) connects the data to the constructor.

## Registration happens once, at startup

All registrations run **at startup**, before the game loop ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) —
part of `GameInit`'s one-time construction. After startup the family lists are fixed and complete, and the rest
of the game constructs objects by looking them up ([C33.3](03-construction.md)). So the registry is built once
and queried forever — the same "build tables once, query many times" pattern as the vault
([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) and geometry directories
([C8.1](../C8-Geometry-Solids/01-solidlist-container.md)).

## RE implications

- **A registration = intern call + list-head link** — find the list-head write to confirm a registration
  ([C33.6](06-using-registry.md)).
- **The registration gives you the constructor and vtable** — entry points to the class's behaviour
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **The family = the role** ([C32.2](../C32-Runtime-Class-System/02-five-roles.md)) — which list-head tells you
  what the class is.
- **Registrations run at startup** — find them in `GameInit`'s construction sequence
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)).

---

### Key takeaways

- A **factory registration** binds name (key), constructor, vtable, and family list-head.
- It's distinguished from a plain hash by the **link onto a family list-head** — the registration's signature.
- This is the **factory registry** pattern: create objects by name, decoupled, extensible, data-driven.
- The shared key ([C32.4](../C32-Runtime-Class-System/04-registration.md)) lets vault data name a class the
  registry builds.
- Registrations run **once at startup**; the family lists are then fixed and queried for the rest of the game.

**Continue:** [C33.3 — Constructing an object](03-construction.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md)
