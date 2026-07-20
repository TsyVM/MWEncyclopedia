# C13.6 — Retuning a Car Safely

> **The one-sentence version:** to retune a car, resolve the field to see whether it's an override or
> inherited, then either overwrite the override in place or add one — writing `Float` bytes, keeping the
> record's size unchanged, and choosing per-car vs behavior-wide vs `default` scope deliberately.

[← C13.5 — The performance bars](05-performance-bars.md) · [Chapter 13 hub](C13-Vault-CarTuning.md) ·
[Next: Chapter 14 — Pursuit, Surfaces & Gameplay →](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)

---

## Decide the scope first

Before touching a byte, decide *which cars* your change should affect — inheritance makes this a real choice
([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)):

| Goal | Edit | Effect |
|---|---|---|
| Retune **one car** | that car's override of the field | only that car |
| Retune a **whole class** (all player cars) | the behavior collection (`EngineRacer`, …) | every car using it that doesn't override |
| Change a **global baseline** | the field in `default` | every collection that inherits it |

Picking the wrong scope is the most common retuning mistake: editing `EngineRacer` to fix one car changes
*all* player cars that share it. Resolve first ([C13.3](03-reading-performance.md)) to see where the value
currently comes from.

## The safe edit: overwrite an existing override

If resolution reports the field is an **override** in the collection you're editing, its `Float` bytes are
already there — overwrite them in place ([C12.6](../C12-Reflection-Schema/06-writing-values.md)):

```python
def retune(buf, value_off, new_float):
    buf[value_off : value_off+4] = struct.pack("<f", new_float)   # Float, 4 bytes, in place
```

No size changes, so the record, the `NtaD` directory ([C11.5](../C11-Attribute-Vaults/05-trailer-blocks.md)),
and every offset stay valid. This is the reliable retune and the one to prefer.

## Adding an override (repack)

If the field is currently **inherited**, changing it for one car means *adding* an override triple, which grows
the record and triggers the vault repack ([C12.6](../C12-Reflection-Schema/06-writing-values.md)): mint the
field id, insert `{id, floatValue}`, bump the field count/size, fix the record and `NtaD` counts. More work —
so if you're changing many cars the same way, editing the shared behavior in place is often simpler than adding
an override to each.

## Tune by intent, not by number

Use the value→knob map ([C13.4](04-value-to-sim.md)) so your edits match your intent:

- **More acceleration** → engine power/response fields (`EngineRacer`).
- **More top speed** → gearing/top-speed fields — and accept softer acceleration.
- **Sharper cornering** → grip fields (`SuspensionRacer`).
- **Stronger NOS** → nitrous fields.

Change one category at a time and test, so you can attribute the result. Remember the bars will re-summarise
automatically ([C13.5](05-performance-bars.md)); you tune fields, not bars.

## Keep it valid

The vault's editing rules apply unchanged:

- **Write the declared type.** Tuning is `Float`; write four float bytes, never integer bytes into a float
  field ([C12.2](../C12-Reflection-Schema/02-schema-map.md)).
- **Match width in place.** A fixed-width `Float` keeps its 4 bytes; nothing downstream moves.
- **Never emit `0xEFFECADD`** as a value ([C11.4](../C11-Attribute-Vaults/04-data-records.md)).
- **Verify by re-resolving.** Read the field back through resolution and confirm it returns your value from the
  collection you intended ([C12.5](../C12-Reflection-Schema/05-resolving-values.md)).

## Sanity-test in motion

Numbers that look right can still feel wrong, so finish at the wheel: after a retune, drive the car and check
the change matches the prediction from [C13.4](04-value-to-sim.md). A power increase that doesn't accelerate
harder, or a grip increase that still slides, usually means you edited the wrong field, the wrong scope, or a
value that another knob caps. The stopwatch and the feel are the final verification the file cannot give you.

---

### Key takeaways

- Decide **scope** first: per-car override, behavior-wide, or `default` — resolve to see where the value comes
  from.
- Overwriting an existing override in place (4 `Float` bytes) is the safe, no-repack retune.
- Adding an override to an inherited field is a repack; for many cars, editing the shared behavior may be
  simpler.
- Tune by intent using the knob map (power→accel, gearing→top speed, grip→cornering), one category at a time.
- Keep types/width valid, never emit the sentinel, verify by re-resolving, and confirm the feel on the road.

**Continue:** [Chapter 14 — Vault Categories: Pursuit, Surfaces & Gameplay](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md) ·
[Chapter 13 hub](C13-Vault-CarTuning.md)
