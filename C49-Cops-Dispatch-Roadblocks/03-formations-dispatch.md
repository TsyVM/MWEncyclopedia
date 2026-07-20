# C49.3 — Formations & Dispatch

> **The one-sentence version:** the fleet's size and mix are set by two Heat-indexed vault tables —
> `CopCountRecord` (×22, how many cops) and `CopFormationRecord` (×22, which shells and how arranged) — with the
> Leader/Heavy/AirSupport strategies authorising heavier dispatch, all fulfilled by `UpdateSpawnRequests`.

[← C49.2 — The cop roster](02-cop-roster.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md) ·
[Next: C49.4 — Roadblocks →](04-roadblocks.md)

---

## Two tables: count and formation

The fleet's composition is entirely **data-driven** by two heavily-referenced vault records, indexed by Heat
([Chapter 48](../C48-Pursuit-Heat/02-heat.md)):

- **`CopCountRecord`** (`0xFCAA46E2`, ×22) — **how many** cops to keep on you at each Heat. The target the fleet
  maintains.
- **`CopFormationRecord`** (`0xB5A53D76`, ×22) — **which shells** ([C49.2](02-cop-roster.md)) and **how they're
  arranged** around you (the pursuit formation).

That each appears **22 times** suggests a rich per-Heat (and per-context) set of rows — the difficulty curve
encoded as table data. `AICopManager` reads the row for the current Heat, gets the target count and formation, and
`UpdateSpawnRequests` ([C49.1](01-fleet-manager.md)) makes the actual fleet match it.

> ✅ *Verified:* `rh("CopCountRecord")=0xFCAA46E2` (×22) and `rh("CopFormationRecord")=0xB5A53D76` (×22) are vault
> keys in `GLOBAL/attributes.bin` — the Heat-indexed dispatch tables.

## Count: maintaining the target

`CopCountRecord` sets the **target number** of cops for the Heat, and the fleet works to maintain it
([C49.1](01-fleet-manager.md)):

- **Below target** (you've lost or disabled cops) → `UpdateSpawnRequests` queues new spawns to top up.
- **At target** → no new spawns; the roster is stable.
- **Cop lost** (disabled, [Chapter 45](../C45-Damage-Deformation/05-cop-damage.md), or shaken) → the count drops,
  triggering a replacement spawn.

So the pursuit maintains a *pressure* — a steady number of cops keyed to Heat. Disabling a cruiser
([Chapter 45](../C45-Damage-Deformation/05-cop-damage.md)) thins the pack only *temporarily*; the count table drives
a replacement, unless you lower the Heat ([Chapter 48](../C48-Pursuit-Heat/04-bust-evade.md)) or escape. This is why
you can't win a pursuit by disabling cops alone — the count table just spawns more, up to the Heat's cap.

## Formation: how they arrange

`CopFormationRecord` sets **how the cops arrange around you** — the *tactics of positioning*:

- **Which shells** ([C49.2](02-cop-roster.md)) — the mix (basic cruisers, SUVs, the chopper) for this Heat.
- **The formation** — how they distribute: some behind (chasing), some flanking, some ahead (to box or
  roadblock, [C49.4](04-roadblocks.md)). A coordinated arrangement, not a disorganised swarm.

The formation is what makes high-Heat pursuits feel *coordinated* — the cops don't just all chase from behind; they
spread to cut you off, flank, and pincer. `CopFormationRecord` is the data that choreographs this. Combined with
the aggressive goals ([Chapter 48](../C48-Pursuit-Heat/03-escalation-ladder.md)), a high-Heat formation is a
genuine tactical net: cars boxing, ramming, and blocking in concert.

> 🟡 *Reasoned:* the interpretation of `CopFormationRecord` as positioning/choreography (flanking, boxing) is the
> natural reading of a "formation" record paired with the count table and the aggressive goals; the exact formation
> geometry is the vault row contents. The records and their ×22 counts are verified.

## The support strategies

At higher Heat, heavier dispatch is authorised by **reinforcement strategies** — a tiered escalation of *what kind
of support* is allowed:

- **Leader** — the base coordination: a lead cop directing the pursuit formation.
- **Heavy** — authorises the heavy units (`copsuv`/Rhinos, [C49.2](02-cop-roster.md)) — the ramming SUVs.
- **AirSupport** — authorises the **helicopter** (`copheli`, [Chapter 48](../C48-Pursuit-Heat/03-escalation-ladder.md))
  — air tracking you can't shake on the ground.

So the strategies are the *gates* on the roster: low Heat gets Leader (basic coordination), higher Heat unlocks
Heavy (SUVs) and then AirSupport (chopper). This tiering is why specific threats appear at specific Heat levels —
the Rhino at Heat 3–4, the chopper at 4–5. The strategies authorise; the count/formation tables supply; the fleet
manager spawns. Three layers of data-driven escalation, all indexed by the one Heat scalar
([Chapter 48](../C48-Pursuit-Heat/02-heat.md)).

## Dispatch is data-over-code

The whole dispatch system embodies the pursuit's **data-over-code** principle
([Chapter 48](../C48-Pursuit-Heat/05-reading-pursuit.md)):

- **The code is fixed** — `AICopManager` reads a Heat, indexes the tables, and spawns to match
  ([C49.1](01-fleet-manager.md)). One routine.
- **The difficulty is data** — `CopCountRecord`, `CopFormationRecord`, and the strategy gates are all vault rows.
  Change them and the whole cop difficulty changes.

So tuning "how many cops, how tough, how coordinated, at what Heat" is editing *tables*, not code. A designer makes
Heat 4 send six SUVs and a chopper in a boxing formation by editing the Heat-4 rows — no engine change. This is why
the pursuit's difficulty is so tunable, and why the ×22 record counts matter: they're the *dials* of the manhunt's
intensity, all keyed to Heat.

## RE implications

- **Two dispatch tables** — `CopCountRecord` (×22, how many) and `CopFormationRecord` (×22, which/how) — indexed by
  Heat.
- **Count maintains a target** — disabled cops are replaced up to the Heat cap; you can't win by disabling alone.
- **Formation choreographs** — flanking, boxing, pincering — coordinated, not a swarm.
- **Strategies gate the roster** — Leader → Heavy (SUVs) → AirSupport (chopper) by Heat.

---

### Key takeaways

- The fleet's composition is **two Heat-indexed vault tables**: `CopCountRecord` (×22, **how many** cops) and
  `CopFormationRecord` (×22, **which shells and how arranged**).
- **Count maintains a target** — the fleet tops up to the Heat's number; disabling cops thins the pack only
  temporarily (replacements spawn up to the cap).
- **Formation choreographs** the cops — flanking, boxing, and pincering, not a disorganised chase — which makes
  high-Heat pursuits feel coordinated.
- **Reinforcement strategies** gate the roster by Heat — **Leader** (coordination) → **Heavy** (SUVs/Rhinos) →
  **AirSupport** (helicopter).
- Dispatch is **data-over-code** — the difficulty is table rows (the ×22 records are the dials); the engine just
  reads Heat and spawns to match.

**Continue:** [C49.4 — Roadblocks](04-roadblocks.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md)
