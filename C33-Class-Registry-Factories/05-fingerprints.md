# C33.5 — Sizes & VTables as Fingerprints

> **The one-sentence version:** a class's object size and vtable method count are its fingerprint — enough to
> gauge its weight and often identify it — so `AIVehicleCopCar` (1964 bytes, 324 methods) reads as a
> heavyweight actor and a tiny connector as a wire, before any code.

[← C33.4 — The class reference](04-class-reference.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md) ·
[Next: C33.6 — Using the registry in RE →](06-using-registry.md)

---

## Two measurements, most of the story

Every catalogue entry ([C33.4](04-class-reference.md)) carries two measurements that, together, characterise a
class:

- **Object size** — how many bytes an instance occupies ([C32.5](../C32-Runtime-Class-System/05-object-model.md)).
  This measures **state**: how much the object remembers.
- **VTable method count** — how many virtual methods the class implements
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)). This measures **behaviour**: how much it can do.

State + behaviour is most of what a class *is*. A big object with many methods is a heavyweight actor; a small
object with few methods is a lightweight helper or wire.

## The extremes

The catalogue's range makes the fingerprint concrete:

- **`AIVehicleCopCar` — 1964 bytes, 324 vtable methods.** The largest and most method-rich registered class. A
  cop car is simultaneously an **entity** (transform, body), an **AI** (goals/actions,
  [Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), and a **physics** object — so it accumulates
  state and behaviour from all three. Its fingerprint alone says "this is the game's most complex actor."
- **A connector — a few dozen bytes, a handful of methods.** A thin wire
  ([C32.2](../C32-Runtime-Class-System/02-five-roles.md)) that moves data across a boundary; its small
  fingerprint says "this does almost nothing on its own."

Between these lie the mechanics (moderate — a component computes one aspect,
[Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)), the AI goals/actions (small-to-moderate — a
decision unit), and the managers (moderate-to-large — they own populations).

> ✅ *Verified (archive Discovery 10):* measured sizes and vtable method counts across the catalogue —
> `AIVehicleCopCar` = 1964 B / 324 methods, the largest.
> 🟡 *Reasoned:* the interpretation (big = heavyweight actor, small = wire) follows from role +
> measurements; the measurements themselves are verified.

## Fingerprints identify classes

The size + method count is often enough to **identify** a class from an object in memory, even without its name:

- **Match the vtable pointer** ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) — the value at object
  `+0x00` uniquely identifies the class if you have the vtable catalogue.
- **Match the size** — an object's byte size narrows the candidates.
- **Match the method count** — the vtable's length distinguishes classes of similar size.

So an unknown object can be identified: read its vtable pointer and size, look them up in the catalogue
([C33.4](04-class-reference.md)). This is how you name a live object during runtime RE
([C33.6](06-using-registry.md)).

## Size measures the data side too

An object's size also tells you about the **data** that configures it
([C32.1](../C32-Runtime-Class-System/01-data-to-objects.md)): a class with many fields has many configurable
values, often mirrored by a large vault collection
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). So a big size hints at a rich data schema —
`AIVehicleCopCar`'s 1964 bytes correspond to the many tuning and state values a cop car carries. The class size
and the vault schema are two views of the same complexity.

## RE implications

- **Fingerprint = size + method count** — gauge a class's weight before reading code.
- **Identify objects** by vtable pointer + size ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **Big fingerprint = start here** — the heavyweight classes (cop cars, managers) are where behaviour
  concentrates.
- **Size hints at the data schema** — a large object often has a large vault collection
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).

---

### Key takeaways

- A class's **size** (state) and **vtable method count** (behaviour) are its fingerprint — most of its story.
- Extremes: `AIVehicleCopCar` (1964 B / 324 methods, heaviest — entity + AI + physics) vs a tiny connector
  (a wire).
- Fingerprints **identify** classes from objects in memory (vtable pointer + size + method count).
- Object size also hints at the **data schema** that configures it (large object ↔ large vault collection).
- Use fingerprints to gauge, identify, and prioritise classes for RE.

**Continue:** [C33.6 — Using the registry in RE](06-using-registry.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md)
