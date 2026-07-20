# C76.2 — Recovering a Schema

> **The one-sentence version:** a *schema* — the field→offset→type map — is the hard part, and the vault's is
> *code-driven*: `speed.exe` holds a 28-name class table (`0x4ADD1C`) whose classes *self-register* at startup onto a
> global list, so the map is *built by registration code*, not stored as a table to parse — which is why the vault
> took a hash breakthrough to crack.

[← C76.1 — Identifying unknown data](01-identifying-data.md) · [Chapter 76 hub](C76-Advanced-RE.md) ·
[Next: C76.3 — Static vs. dynamic recovery →](03-static-vs-dynamic.md)

---

## The schema is the hard part

Identifying the *shape* of a format ([C76.1](01-identifying-data.md)) is the easy half; the hard half is the
**schema** — the map of *which field lives at which offset, with which type*, inside each record. For the vault
([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)), the shape was clear (a string table + a 4,732-
record array, [C76.1](01-identifying-data.md)), but the schema — *what each record's bytes mean* — was genuinely
**XL**. Understanding *why* it was hard is the lesson: the schema isn't *stored*; it's *built by code*.

## The class name table

The starting thread is a **class-name table** in the executable. At file offset `0x4ADD1C`, `speed.exe` holds 28
reflection class names as inline NUL-padded strings:

```
pvehicle  chopperspecs  damagespecs  rigidbodyspecs  RBVehicle  RBTrailer  RBCop
EffectsVehicle  EffectsCar  EffectsFragment  DamageRacer  DamageHeli  DamageCopCar
SuspensionTraffic  SuspensionTrailer  EngineRacer  SimpleChopper  EngineTraffic
DrawHeli  DrawTraffic  DrawNISCar  DrawCopCar  DrawRaceCar  SoundTraffic  SoundCop
SoundRacer  SoundHeli  SpikeStrip
```

These are the *simulation classes* whose fields the vault serialises — the car, the cop, the trailer, their engine,
suspension, damage, effects, sound, and draw sub-objects ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).
So the class table is the *index* of what the schema must cover: 28 classes, each with a field set to recover.

> ✅ *Verified:* the 28-name class table is at file offset `0x4ADD1C` in `speed.exe` — inline NUL-padded strings
> (`pvehicle\0\0\0\0chopperspecs\0\0\0\0…`), confirmed by direct read.

## Code-driven registration

Here's why the schema is hard: it's **not a static table you can parse** — it's *assembled at startup by registration
code*. Each class has a tiny static-initialiser thunk in the executable that, at load time, **pushes the class onto a
global linked list**: it interns the class name (through a shared hash/intern function), stores a constructor pointer
and the previous list head, and updates the global head. This is the classic C++ idiom — *static objects register
themselves onto a list walked once at startup* ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) — and it
means:

- **The field→offset→type map exists only after the registration code runs** — it's built in memory, not laid out in
  a file or a static data table.
- **Recovering it statically means reading the *code*** — tracing the registration thunks and the constructors that
  add fields, not parsing a table ([C76.3](03-static-vs-dynamic.md)).
- **The registration is code-driven** — so there's no shortcut of "find the schema table"; the schema *is* the
  behaviour of the startup code.

This is the crux of an XL schema: when the format's structure is *emergent from code* rather than *stored as data*,
static recovery is a disassembly problem, not a parsing one — far harder, and why the vault resisted so long.

> ✅ *Verified:* each class self-registers via a static-init thunk that pushes it onto a global list (interned through
> a shared function, list head at a fixed global) — the "static object self-registers, list walked at startup" idiom,
> confirmed by disassembly of the thunks.

## The hash breakthrough

If the schema is code-driven, how was the vault cracked? By a **hash breakthrough** on the record *keys*
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)). Each `NtaD` record is keyed by a 4-byte value; the
question was *what those keys are*. An early attempt hashed the `ErtS` names with Joaat and Bin
([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)) and got **<0.2%** matches — noise — and
*wrongly* concluded the keys weren't inline name hashes ([C76.4](04-building-readers.md)). The correction: the vault
uses a *third* hash — **`lookup2` with seed `0xABCDEF00`** (the reflection hash,
[Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) — under which **66.8%** of the keys resolve to
`ErtS` names. That single change (right hash) turned the record keys from opaque to *nameable*, unlocking the value
model ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) — inline `{field, value, type}` triples —
without ever fully tracing the registration code.

So the schema was recovered *around* the hard part: not by reading the code that builds the field map, but by
*matching the record keys to names* via the right hash, then reading the values. This is a general lesson
([C76.5](05-advanced-method.md)) — when the *structure-building code* is XL, attack the *data* from a different angle
(the keys), and let a statistical match (66.8% vs. 0.2%) confirm you've found the right one.

> ✅ *Verified:* the `lookup2` seed `0xABCDEF00` appears ×50 in `speed.exe`; the vault record keys resolve as
> `lookup2`/`0xABCDEF00` hashes of `ErtS` names at ~66.8% (vs. <0.2% under Joaat/Bin) — the hash that cracked the
> vault ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).

## RE implications

- **The schema is the hard part** — the field→offset→type map, vs. the easy *shape* ([C76.1](01-identifying-data.md)).
- **The class table** — `0x4ADD1C`, 28 sim-class names; the index of what the schema covers.
- **Code-driven registration** — the map is *built by startup code* (self-registering classes), not a static table —
  so static recovery is a disassembly problem.
- **The hash breakthrough** — record keys are `lookup2`/`0xABCDEF00` hashes of names (66.8% resolve); attack the keys,
  not the code.

---

### Key takeaways

- The **schema** — the field→offset→type map — is the hard half of RE (the *shape* is easy,
  [C76.1](01-identifying-data.md)); the vault's was genuinely **XL**.
- The starting thread is the **28-name class table** (`0x4ADD1C`) — the sim classes (`pvehicle`, `RBCop`,
  `EngineRacer`, …) whose fields the vault serialises.
- The schema is **code-driven** — each class **self-registers** onto a global list at startup
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)), so the field map is **built by code, not stored as a
  table** — making static recovery a **disassembly** problem, not a parsing one.
- The vault was cracked by a **hash breakthrough on the keys** — they're **`lookup2`/`0xABCDEF00`** hashes of `ErtS`
  names (**66.8%** resolve, vs. <0.2% under the wrong hash), which made records **nameable** without fully tracing the
  registration code.
- The lesson: when the **structure-building code is XL**, attack the **data from another angle** (the keys) and let a
  **statistical match** confirm the right hypothesis ([C76.4](04-building-readers.md)).

**Continue:** [C76.3 — Static vs. dynamic recovery](03-static-vs-dynamic.md) · [Chapter 76 hub](C76-Advanced-RE.md)
