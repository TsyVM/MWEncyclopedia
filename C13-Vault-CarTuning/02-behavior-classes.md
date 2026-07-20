# C13.2 — Physics Behavior Classes

> **The one-sentence version:** MW models a car as a set of swappable behavior collections — an engine, a
> suspension, a damage model, a sound model — each a vault record of mostly-`Float` tuning fields inheriting
> from `default`, so tuning a car means tuning its behaviors.

[← C13.1 — The car-tuning collection map](01-collection-map.md) · [Chapter 13 hub](C13-Vault-CarTuning.md) ·
[Next: C13.3 — Reading a car's performance →](03-reading-performance.md)

---

## The four player behaviors

The player car (`…Racer`) is composed from four behavior collections, each modelling one aspect of the
vehicle:

- **`EngineRacer`** (`0xB2809518`) — the **powertrain**: how throttle becomes force, the power curve, response,
  and the effective top-speed/gearing behaviour.
- **`SuspensionRacer`** (`0x6209E06A`) — the **chassis dynamics**: grip, weight transfer, body roll, how the
  car turns and holds the road.
- **`DamageRacer`** — the **collision model**: how impacts affect handling and the car's state.
- **`SoundRacer`** — the **audio model**: engine note, shift sounds (its fields drive sound banks like the
  `GEAR_*` and `Nitrous_*` entries in the string table).

Each is a collection ([C13.1](01-collection-map.md)) whose record holds the tuning fields for that aspect,
resolved through `default` ([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)).

## A behavior is a record of Float knobs

Decoding the `EngineRacer` record ([C12.3](../C12-Reflection-Schema/03-value-triple.md)) shows the expected
shape: a collection hash, the `default` parent reference, and a series of `{field, value, type}` triples whose
values are `Float` tuning constants. The engine behavior's fields govern the powertrain; the suspension
behavior's govern grip and body motion. You read them exactly as any vault fields — resolve, then decode as
`Float` ([C13.3](03-reading-performance.md)).

> ✅ *Verified:* `EngineRacer` and `SuspensionRacer` are real records inheriting from `default`, carrying
> `Float`-typed fields.
> 🟡 *Reasoned:* the mapping of each specific field to a named physical quantity (which `Float` is "peak
> power," which is "gear ratio") requires the full schema field-name map; the *structure* (behaviors as
> Float-field collections) is verified, individual field semantics are established by resolving names from the
> string table and cross-referencing values.

## Why "behaviors" and not "stats"

The behavior decomposition is more than bookkeeping — it mirrors how the simulation is built:

- **The simulation has modules.** The engine solver, the suspension/tire solver, and the damage system are
  separate code; each reads its own behavior collection. The data layout follows the code layout.
- **Behaviors are swappable.** Because the engine model is a separate collection, a car can point at a
  different engine behavior (racer vs traffic vs cop) without changing anything else — the simulation just
  binds a different record.
- **Agents share components.** `EngineRacer`, `EngineTraffic`, and the cop engine are the *same model* with
  *different tuning*, so civilian and police cars reuse the player's physics code at their own settings.

This is the vault's composition philosophy ([C12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) applied
to physics: small, named, inheriting pieces rather than one monolith.

## Reading a behavior

```python
def read_behavior(vault, behavior_name, field_names, schema):
    ch = reflection_hash(behavior_name)                # e.g. "EngineRacer"
    out = {}
    for fname in field_names:
        value, ftype, src = resolve(vault, ch, reflection_hash(fname), schema)  # C12.5
        out[fname] = (value, ftype, "override" if src == ch else "inherited")
    return out
```

The `override`/`inherited` tag matters: it tells you whether *this* behavior set the value or took it from
`default`, which is exactly what you need before editing ([C13.6](06-retuning.md)).

## Editing implications

- **Edit the behavior for a class-wide change.** Change `EngineRacer` and every player car using it (that
  doesn't override the field) is affected.
- **Keep types.** Behavior fields are `Float`; write float bytes ([C12.6](../C12-Reflection-Schema/06-writing-values.md)).
- **Mind the audio link.** `SoundRacer` fields reference sound banks (`GEAR_*`, `Nitrous_*`); changing them
  changes what you hear, not how the car drives — the driving fields live in `EngineRacer`/`SuspensionRacer`.

---

### Key takeaways

- The player car is four behaviors: `EngineRacer` (powertrain), `SuspensionRacer` (chassis), `DamageRacer`
  (collision), `SoundRacer` (audio).
- Each behavior is a vault record of mostly-`Float` tuning fields inheriting from `default`.
- The decomposition mirrors the simulation's modules and lets agents share components at different tunings.
- Read a behavior by resolving its fields; the override/inherited tag guides editing.
- Edit a behavior for a class-wide change; keep field types; audio lives in `SoundRacer`, handling in
  engine/suspension.

**Continue:** [C13.3 — Reading a car's performance](03-reading-performance.md) · [Chapter 13 hub](C13-Vault-CarTuning.md)
