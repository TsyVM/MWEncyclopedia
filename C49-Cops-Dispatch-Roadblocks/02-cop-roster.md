# C49.2 — The Cop Roster

> **The one-sentence version:** the cops come in a verified roster of shells — `cop1` (basic cruiser),
> `copmidsize`, `copsport`, `copsuv`/`copsuvpatrol` (SUVs/Rhinos), `copcross`, the named `copgto`/`COPGTO`, and
> `copheli` (helicopter) — which Heat selects from, light units low and heavy units (plus the chopper) high.

[← C49.1 — AICopManager & the OnTask pipeline](01-fleet-manager.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md) ·
[Next: C49.3 — Formations & dispatch →](03-formations-dispatch.md)

---

## The roster of shells

The cops you face aren't one car — they're a **roster of vehicle shells**, verified as strings in `speed.exe`,
each a distinct cruiser used at different Heat levels ([Chapter 48](../C48-Pursuit-Heat/02-heat.md)):

| Shell | Role | Heat tier |
|---|---|---|
| `cop1` | the basic city cruiser | low |
| `copmidsize` | a mid-size sedan | low–mid |
| `copsport` | a faster sport cruiser | mid |
| `copsuv` / `copsuvpatrol` | the SUV / Rhino (heavy) | mid–high |
| `copcross` | a cross-country / undercover unit | mid–high |
| `copgto` / `COPGTO` | the named GTO pursuit unit | high (special) |
| `copheli` | the police helicopter | high |
| `copghost` / `copgtoghost` | "ghost" variants | special |

> ✅ *Verified:* the cop shells `cop1`, `copmidsize`, `copsport`, `copsuv`, `copsuvpatrol`, `copcross`,
> `copgto`/`COPGTO`, `copheli`, and the ghost variants (`copghost`, `copgtoghost`) are present as strings in
> `speed.exe`. `copsuv` (×16), `copheli` (×15), and `copcross` (×7) appear as vault keys in `attributes.bin`.

## Heat selects the tier

The roster maps onto the Heat escalation ([Chapter 48](../C48-Pursuit-Heat/02-heat.md)) — `CopFormationRecord`
([C49.3](03-formations-dispatch.md)) selects which shells are dispatched at each Heat:

- **Low Heat (1–2)** — `cop1` and `copmidsize`: basic sedans, a couple of them. Easy to outrun.
- **Mid Heat (3)** — `copsport` and the first SUVs (`copsuv`): faster, tougher, more of them.
- **High Heat (4–5)** — heavy `copsuv`/`copsuvpatrol` (the Rhinos that ram),  `copcross`, the named `copgto`, and
  the **helicopter** (`copheli`) overhead. A swarm.

So climbing Heat *walks up the roster*: the cars chasing you get faster and heavier, and new unit types (SUVs,
chopper) appear. The felt escalation — from a lone cruiser to a Rhino-and-chopper manhunt — is this roster being
indexed higher ([Chapter 48](../C48-Pursuit-Heat/02-heat.md)). The reference counts (`copsuv` ×16, `copheli` ×15)
reflect how central the mid/high-tier units are to the pursuit data.

## The named units: COPGTO

Most shells are *generic* (any `cop1` is interchangeable), but a few are **named units** — most notably
**`copgto`/`COPGTO`**, the Pontiac GTO cop car. Named units are special:

- **Distinct vehicles.** A named unit like the GTO cruiser is a specific, recognisable car
  ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md) — registered by name, not mass-produced),
  used for high-Heat or story-significant pursuit moments.
- **The "ghost" variants** (`copghost`, `copgtoghost`) are likely the *ghost/replay* or *spawn-transition* forms —
  a cop rendered specially (transparent, or during a spawn effect) before becoming a full pursuer.

So the roster has both a *generic* body (the interchangeable cruisers spawned in bulk against the count table,
[C49.3](03-formations-dispatch.md)) and a few *named* bodies (the GTO) for special cases. This mix — mass-produced
plus named — is the same pattern as the object model's factory-vs-named-node distinction
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)): most cops are anonymous units,
a few are characters.

> 🟡 *Reasoned:* the interpretation of `copgto` as a named high-Heat unit and the `*ghost` variants as spawn/replay
> forms is inferred from the naming and the named-vs-generic object model; the exact per-shell usage is pursuit
> vault data. The shell strings and their vault-key counts are verified.

## Shells vs. brains

An important distinction: a cop *shell* ([above](#the-roster-of-shells)) is the *vehicle* (the car body,
[Chapter 41](../C41-Physics-RigidBody/01-rigidbody-tree.md) `RBCop`), while the cop *brain* is `AIVehicleCopCar`
([Chapter 47](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md), 324 methods). Every shell — `cop1`, `copsuv`,
`copgto` — is driven by the *same* `AIVehicleCopCar` brain; they differ in their *car* (speed, mass, toughness),
not their *intelligence*. So a Rhino (`copsuv`) and a basic cruiser (`cop1`) think alike (same pursuit brain and
goals, [Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) but *drive* differently (the Rhino is heavier
and rams harder). This separation — one brain, many shells — is why Heat can throw tougher *cars* at you without
needing new *AI*: the escalation is in the vehicles, over a shared cop mind.

## RE implications

- **The cop roster is a verified set of shells** — `cop1`, `copmidsize`, `copsport`, `copsuv`, `copcross`,
  `copgto`, `copheli`, ghosts.
- **Heat selects the tier** — light cruisers low, SUVs/chopper high (`CopFormationRecord`,
  [C49.3](03-formations-dispatch.md)).
- **Named units** (`copgto`) vs. **generic** cruisers — the factory-vs-named-node pattern
  ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).
- **Shells vs. brains** — every shell is driven by the same `AIVehicleCopCar` brain; escalation is in the *cars*.

---

### Key takeaways

- The cops are a **verified roster of shells** — `cop1`, `copmidsize`, `copsport`, `copsuv`/`copsuvpatrol`,
  `copcross`, `copgto`/`COPGTO`, `copheli`, and ghost variants.
- **Heat selects the tier** (`CopFormationRecord`) — basic sedans low, SUVs/Rhinos and the chopper high; climbing
  Heat walks up the roster.
- Most shells are **generic** (mass-produced cruisers); a few are **named units** (the GTO), the same
  factory-vs-named-node split as the object model.
- **Shells vs. brains** — every cop car, from `cop1` to `copsuv`, is driven by the **same `AIVehicleCopCar`
  brain**; they differ in the *vehicle*, not the *mind*.
- Escalation throws **tougher cars** at you over a **shared cop AI** — new vehicles, same intelligence.

**Continue:** [C49.3 — Formations & dispatch](03-formations-dispatch.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md)
