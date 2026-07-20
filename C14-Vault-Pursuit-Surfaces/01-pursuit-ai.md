# C14.1 — The Pursuit & AI Vault

> **The one-sentence version:** the police are attribute-driven — `AIPursuit` tunes chase behaviour,
> `AICopManager` governs spawning and reinforcements, and the 71-strong `COP*` roster defines each police
> vehicle with the same behavior-collection model as player cars.

[← Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md) · [Next: C14.2 — The heat & bounty system →](02-heat-bounty.md)

---

## The three layers of the police

Most Wanted's pursuit is built from three vault layers, each a verified collection:

| Layer | Collection | Hash | Role |
|---|---|---|---|
| **Behaviour** | `AIPursuit` | `0x1F319B62` | how cops chase, ram, box in, coordinate |
| **Management** | `AICopManager` | `0x5DB210B6` | how many cops, when reinforcements/roadblocks arrive |
| **Vehicles** | `COP*` roster | (71 collections) | each police car's stats (`COPMIDSIZE`, `COPSPORT`, `COPHELI`, …) |

Together they answer "who chases you, how many, and how hard." Tuning any layer changes the pursuit's
character — `AIPursuit` its aggression, `AICopManager` its scale, the roster its hardware.

## AIPursuit — the chase behaviour

`AIPursuit` (`0x1F319B62`) holds the tuning for *how* police pursue: pursuit tactics, ramming aggression,
formation/coordination, and how relentlessly they stick to you. Like every collection it inherits from
`default` ([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)) and overrides the fields that define
pursuit behaviour. Raising aggression fields makes cops ram and box more; lowering them makes them cautious.

## AICopManager — the escalation controller

`AICopManager` (`0x5DB210B6`) governs the *scale* of the response: how many police units are active, spawn
rates, when reinforcements and roadblocks/helicopter deploy. It is the dial between "a lone cruiser" and "a
full multi-unit pursuit," and it works hand-in-hand with the heat system
([C14.2](02-heat-bounty.md)) that decides which settings apply at each notoriety level.

## The COP roster — police vehicles

The 71 `cop`-prefixed collections are police *vehicles*, defined exactly like player cars
([C13.1](../C13-Vault-CarTuning/01-collection-map.md)) — each selects behavior components and overrides tuning:

- `COPMIDSIZE` (`0x7D29EBD1`) — the standard sedan cruiser.
- `COPSPORT` (`0x5339838D`) — the faster sports interceptor.
- `COPSUV` — the heavy SUV (Rhinos).
- `COPHELI` (`0x47C914A3`) — the helicopter.
- plus ghost/henchman/variant collections (`COPGHOST`, `COPSPORTHENCH`, …).

So making the police faster or tougher is retuning the relevant roster collection's behavior fields, the same
operation as retuning a player car ([C13.6](../C13-Vault-CarTuning/06-retuning.md)).

> ✅ *Verified:* `AIPursuit` (0x1F319B62), `AICopManager` (0x5DB210B6), `COPMIDSIZE` (0x7D29EBD1),
> `COPSPORT` (0x5339838D), and `COPHELI` (0x47C914A3) are present collections; 71 `cop`-prefixed names exist
> in the string table.
> 🟡 *Reasoned:* the specific behaviour each field controls (which `Float` is "ram aggression") follows from
> resolving field names and cross-referencing; the collection roles are verified.

## Tuning the pursuit

```python
def pursuit_report(vault, schema, name_map):
    for col in ["AIPursuit", "AICopManager", "COPMIDSIZE", "COPSPORT", "COPHELI"]:
        rec = vault.record_by_hash(reflection_hash(col))     # C12.1
        print(col, "→", labelled_fields(rec, schema, name_map))   # C13.3
```

The pattern is identical to car tuning: find the collection, resolve and label its fields, edit the ones that
match your intent. A "harder pursuit" mod is a combination — more aggressive `AIPursuit`, more units via
`AICopManager`, faster `COP*` vehicles — each a vault edit ([C14.6](06-editing-gameplay.md)).

---

### Key takeaways

- Pursuit has three layers: `AIPursuit` (behaviour), `AICopManager` (scale/spawning), and the `COP*` vehicle
  roster.
- All are verified collections inheriting from `default`; the roster has 71 members.
- `AIPursuit` tunes aggression/tactics; `AICopManager` tunes how many cops and when reinforcements come.
- `COP*` vehicles are defined like player cars — retune them the same way.
- A "harder pursuit" is a combined edit across all three layers.

**Continue:** [C14.2 — The heat & bounty system](02-heat-bounty.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md)
