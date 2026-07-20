# C32.6 — Reading the Class System from the Binary

> **The one-sentence version:** to read the runtime you read `speed.exe` — find a family's registrations via its
> list-head, hash a class name to its key, identify a class by its vtable and size, and classify it by role —
> turning the executable into a legible class catalogue.

[← C32.5 — The object model](05-object-model.md) · [Chapter 32 hub](C32-Runtime-Class-System.md) ·
[Next: Chapter 33 — The Class Registry, Factories & the Class Reference →](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)

---

## The tools change

Reverse-engineering the runtime means reading the **executable**, not file bytes. The workflow
([Chapter 4](../C4-Byte-Level-Toolcraft/C4-Byte-Level-Toolcraft.md) applied to code):

- **Disassemble `speed.exe`** — a 6 MB PE32; use a disassembler (or a scripted capstone pass) to read functions.
- **Find registrations** — code that hashes a name and links onto a family list-head
  ([C32.4](04-registration.md)).
- **Follow vtables** — from a class's constructor to its vtable to its methods
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **Measure** — object size (allocation) and vtable method count ([C32.5](05-object-model.md)).

The disassembly is the primary source; the class system is what you reconstruct from it.

## Anchors: the list-heads

The eleven family list-heads ([C32.3](03-eleven-families.md)) are your entry points. Find the code that
references a list-head, and you've found where that family's classes register:

```
find references to 0x0092C660 (mechanics list-head)
   → each is a class registering as a mechanic
   → from the registration: the class name, its constructor, its vtable
```

Verified, `0x0092C660` has 105 references in `speed.exe` — a rich seam of mechanic registrations. Walking a
list-head's references enumerates a family's classes, giving you the catalogue
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).

## Names are computable keys

Because class names use the reflection hash ([C32.4](04-registration.md)), you can go from a name to its key and
back:

- **Name → key.** `reflection_hash("EngineRacer") = 0xB2809518` ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)) —
  find where that key appears to find the class's registration/use.
- **Key → name.** A key in the binary can be matched against hashes of candidate names (the vault's `ErtS`
  strings, [C11.2](../C11-Attribute-Vaults/02-erts-strings.md), are a dictionary).

This computability ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)) is a gift the asset hash doesn't
give: class and vault names can be recovered/matched by hashing, so a mystery key often resolves to a readable
name.

## The identification method

Putting it together, identifying an unfamiliar class ([C32.2](02-five-roles.md)–[C32.5](05-object-model.md)):

1. **Which family?** — which list-head its registration targets → its role
   ([C32.2](02-five-roles.md), [C32.3](03-eleven-families.md)).
2. **What name?** — from the registration, or by matching its key-hash to a name dictionary.
3. **How big?** — object size ([C32.5](05-object-model.md)) — its state weight.
4. **How much behaviour?** — vtable method count ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) — its
   behaviour weight.
5. **What does it do?** — role + size + vtable shape, then read key methods.

Steps 1–4 are cheap and give most of the story; step 5 is the deep dive when needed.

## Cross-referencing data and code

The runtime RE connects to the data RE via the shared hash ([C32.4](04-registration.md)):

- A **vault collection** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and a **runtime class**
  of the same name are the same entity — decode the vault to learn the data, read the class to learn the code.
- **`EngineRacer`** is both a vault collection (tuning) and a class (0xB2809518) — the data configures the code.

So the file-format chapters and the runtime chapters meet at the class: the vault tells you *what values*, the
class tells you *what the game does with them*.

## RE checklist

- **Start at a list-head** ([C32.3](03-eleven-families.md)) to enumerate a family.
- **Hash names** to keys and match keys to names ([C32.4](04-registration.md)).
- **Fingerprint by size + method count** ([C32.5](05-object-model.md)).
- **Classify by role** ([C32.2](02-five-roles.md)), then read methods ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **Cross-reference the vault** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) for the data side.

---

### Key takeaways

- Read the runtime by disassembling **`speed.exe`**; the class system is reconstructed from it.
- The eleven **list-heads** are anchors — their references enumerate each family's registrations.
- Class **names are computable keys** (reflection hash) — resolve mystery keys to names via a dictionary.
- Identify a class by **family (role) → name → size → method count → methods** — cheap first, deep last.
- The shared hash links **vault (data) and class (code)** of the same name — decode both to understand an
  entity fully.

**Continue:** [Chapter 33 — The Class Registry, Factories & the Class Reference](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md) ·
[Chapter 32 hub](C32-Runtime-Class-System.md)
