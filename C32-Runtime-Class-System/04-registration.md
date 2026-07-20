# C32.4 — Registration: Name → Hash → List-Head

> **The one-sentence version:** at startup each class hashes its name with the reflection hash
> (`lookup2`/`0xABCDEF00`) and links itself onto its family's list-head — the *same* hash the vault uses — so
> the class registry, the vault, and reflection are one hash world.

[← C32.3 — The eleven families](03-eleven-families.md) · [Chapter 32 hub](C32-Runtime-Class-System.md) ·
[Next: C32.5 — The object model →](05-object-model.md)

---

## Startup registration

Before the game runs, every class **registers** itself. The registration, seen in `speed.exe`, does two things:

1. **Hash the class name** with the reflection hash — `lookup2` seeded `0xABCDEF00`
   ([C2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)) — producing the class's key.
2. **Link the class onto its family's list-head** ([C32.3](03-eleven-families.md)) — adding it to the global
   list for its role.

```
class "EngineRacer" at startup:
   key = reflection_hash("EngineRacer") = 0xB2809518     ; call to the hash routine (0x5CC240)
   link {name, key, constructor, vtable} onto mechanics list-head (0x0092C660)
```

After all classes register, the runtime has eleven populated lists ([C32.3](03-eleven-families.md)), each class
findable by its name-hash. This is the reflection system for *code*: classes are self-describing, registered by
name, constructable by hash ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).

## The same hash as the vault

The pivotal fact: the class name-hash is the **same reflection hash** the vault uses for field names
([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)). Verified — `reflection_hash("EngineRacer") =
0xB2809518` and `("SuspensionRacer") = 0x6209E06A`, and these are *identical* to the vault's `EngineRacer` and
`SuspensionRacer` collection hashes ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). This is not a
coincidence — it's a unification:

- **The vault** ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) hashes field/collection names
  with `lookup2`/`0xABCDEF00`.
- **The class registry** (this chapter) hashes class names with the *same* function.
- **So `EngineRacer` the vault collection and `EngineRacer` the runtime class share one hash** — the data and
  the code are keyed the same way.

This is why the vault's `EngineRacer` tuning ([C13.2](../C13-Vault-CarTuning/02-behavior-classes.md)) configures
the runtime's `EngineRacer` class: they are the same name, the same hash, the same entity seen as data and as
code.

> ✅ *Verified:* the class name-hash is `lookup2`/`0xABCDEF00` — `EngineRacer` → `0xB2809518`, `SuspensionRacer`
> → `0x6209E06A`, matching the vault; the hash constants (`0x9E3779B9`, `0xABCDEF00`) are present in
> `speed.exe` (4 and 50 times), and the registration hash site is the reflection-hash routine
> ([C2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)).

## One hash world

The unification extends across the whole engine. The reflection hash (`lookup2`/`0xABCDEF00`) keys:

- **Vault fields and collections** ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).
- **Runtime classes** (this chapter).

— all one namespace. This is distinct from the **asset hash** ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md))
that keys textures and geometry names (not recoverable). So the engine has exactly two hash worlds
([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)), and the reflection one spans *both* data (vault) and
code (classes) — the deepest cross-cutting fact in the book.

## What registration enables

Registering classes by name-hash onto family lists is what makes the runtime **reflective**:

- **Construct by name.** "Make an `EngineRacer`" hashes the name and finds the class to construct
  ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).
- **Data-driven instantiation.** The vault names a class ([C13.2](../C13-Vault-CarTuning/02-behavior-classes.md)),
  the registry constructs it — data selects code by shared hash.
- **Enumerate by family.** Walk a family's list to process all its classes
  ([C32.3](03-eleven-families.md)).

So registration is the mechanism behind data-driven construction: because the vault and the classes share the
hash, naming a behavior in data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) instantiates the
matching class in code.

## RE implications

- **Class-name hashes are computable** — unlike asset hashes, you can hash a class name to find its key
  ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)).
- **The registration site is an anchor** — code hashing a name and linking to a list-head is a class
  registration ([C32.6](06-reading-binary.md)).
- **Vault ↔ class link** — a vault collection and a class of the same name are the same entity
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); use one to understand the other.
- **The hash unifies the book** — reflection hash spans data and code; asset hash spans content.

---

### Key takeaways

- At startup each class **hashes its name** (reflection hash, `lookup2`/`0xABCDEF00`) and **links onto its
  family list-head**.
- The class name-hash is the **same** as the vault's field hash — `EngineRacer` → `0xB2809518` both as class and
  collection (verified).
- This unifies **data (vault) and code (classes)** into one reflection-hash namespace (distinct from the asset
  hash).
- Registration makes the runtime **reflective**: construct-by-name, data-driven instantiation, enumerate-by-
  family.
- Class-name hashes are **computable**; the vault↔class shared name is the same entity as data and code.

**Continue:** [C32.5 — The object model](05-object-model.md) · [Chapter 32 hub](C32-Runtime-Class-System.md)
