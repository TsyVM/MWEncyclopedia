# C13.1 — The Car-Tuning Collection Map

> **The one-sentence version:** a car's driving data is spread across behavior collections — `EngineRacer`,
> `SuspensionRacer`, `DamageRacer`, `SoundRacer` for the player, parallel families for AI/traffic, and a large
> roster of `COP*` police vehicles — each a real vault record you can find by reflection hash.

[← Chapter 13 hub](C13-Vault-CarTuning.md) · [Next: C13.2 — Physics behavior classes →](02-behavior-classes.md)

---

## The map

Car tuning is not one collection; it is a family. Located by reflection hash in the live vault:

| Collection | Hash | Role |
|---|---|---|
| `EngineRacer` | `0xB2809518` | player powertrain model |
| `SuspensionRacer` | `0x6209E06A` | player grip/body dynamics |
| `DamageRacer` | (hashable) | player collision/damage model |
| `SoundRacer` | (hashable) | player engine audio model |
| `Physics` | `0x09900113` | shared physics constants |
| `car` | `0xA13753EB` | car-level collection / base |

Alongside the player `…Racer` family sit parallel families for other agents — `…Traffic` (civilian traffic),
`…Cop` (police) — and the **police roster**: `COPMIDSIZE`, `COPSPORT`, `COPSUV`, `COPHELI`, `COPGTO`,
`COPSPORTHENCH`, and dozens more (the string table holds 71 `cop`-prefixed names). Each is its own collection.

## Reading the map

Two conventions make the family legible once you know them:

- **Suffix = agent class.** `…Racer` is the *player* car; `…Traffic` is civilian; `…Cop`/`COP…` is police.
  The same behavior (engine, suspension) exists per agent class so the simulation can tune player and AI cars
  independently.
- **Prefix/name = component.** `Engine…`, `Suspension…`, `Damage…`, `Sound…` name the *component* a behavior
  models. A full car is the sum of its components' behaviors.

So `EngineRacer` = "the engine model for the player car," `SuspensionCop` = "the suspension model for police
cars," and so on. This orthogonal *component × agent* grid is the whole map.

## Why split this way

Composing a car from independent behaviors, each a vault collection, buys the same flexibility the reflection
system buys everywhere:

- **Independent tuning.** Change the player engine without touching suspension, or make police cars grippier
  without changing their engines.
- **Reuse via inheritance.** Every behavior inherits from `default`
  ([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)); a cop variant overrides only what differs from
  the baseline cop, which overrides only what differs from `default`.
- **Per-car specialisation.** A specific car selects behaviors and overrides individual fields, so the M3 GTR
  and a base sedan can share the engine *model* while differing in its *values*.

## Finding a collection in practice

```python
def find_collection(vault, name):
    h = reflection_hash(name)                 # C12.1
    rec = vault.record_by_hash(h)
    return rec                                 # its overrides; resolve fields via C12.5

for name in ["EngineRacer", "SuspensionRacer", "COPSPORT"]:
    rec = find_collection(vault, name)
    print(name, hex(reflection_hash(name)), "found" if rec else "inherited-only")
```

Because names hash predictably ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)), you go straight from
a behavior name to its record — no scanning. From there, resolve its fields
([C13.3](03-reading-performance.md)).

> ✅ *Verified:* `EngineRacer` (0xB2809518), `SuspensionRacer` (0x6209E06A), `Physics` (0x09900113), and `car`
> (0xA13753EB) are present as records inheriting from `default`; the `…Racer` family and the 71-strong `COP*`
> roster are in the string table.

## Editing implications

- **Pick the right agent class.** Edit `…Racer` for the player, `COP…`/`…Cop` for police — changing the wrong
  family retunes the wrong cars.
- **Pick the right component.** Acceleration lives in the engine behavior; cornering in suspension; crash
  behaviour in damage.
- **Use inheritance for scope** ([C12.6](../C12-Reflection-Schema/06-writing-values.md)): edit a specific car's
  override for one car, or the behavior collection for the whole class.

---

### Key takeaways

- Car tuning is a **component × agent** grid of behavior collections, not one struct.
- Components: `Engine`, `Suspension`, `Damage`, `Sound`; agents: `…Racer` (player), `…Traffic`, `COP…`
  (police).
- Verified records include `EngineRacer` (0xB2809518), `SuspensionRacer` (0x6209E06A), `Physics`, `car`; 71
  `COP*` collections exist.
- The split enables independent tuning, inheritance-based reuse, and per-car specialisation.
- Find any collection by reflection hash, then resolve its fields; edit the family/component/scope that matches
  your intent.

**Continue:** [C13.2 — Physics behavior classes](02-behavior-classes.md) · [Chapter 13 hub](C13-Vault-CarTuning.md)
