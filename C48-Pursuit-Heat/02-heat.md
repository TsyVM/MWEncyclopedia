# C48.2 — Heat: the Escalation Variable

> **The one-sentence version:** Heat is the master intensity scalar of a pursuit — it indexes the vault dispatch
> tables `CopCountRecord` (×22) and `CopFormationRecord` (×22), so each Heat level selects how many cops, what
> mix and formation, which support strategy, and how aggressive the goals may get.

[← C48.1 — The cast](01-the-cast.md) · [Chapter 48 hub](C48-Pursuit-Heat.md) ·
[Next: C48.3 — The escalation ladder →](03-escalation-ladder.md)

---

## Heat is a scalar that indexes tables

**Heat** is the single number that drives a pursuit's intensity (and tracks your career standing). Its role is
mechanically simple and powerful: Heat is a **scalar that indexes the vault dispatch tables**. At each Heat level,
the game reads the corresponding rows of two heavily-used vault records:

- **`CopCountRecord`** (verified ×22 in `attributes.bin`) — **how many** cops to keep on you at this Heat.
- **`CopFormationRecord`** (verified ×22) — **what mix and formation** — sedans at low Heat, SUVs and heavier
  units higher, the chopper higher still.

So raising Heat doesn't run different *code* — it selects *heavier rows* of the same tables. Heat 2 reads a light
row (a couple of sedans); Heat 5 reads a heavy row (many units, SUVs, a helicopter). This table-indexing is the
whole mechanism, which is why the pursuit is "almost entirely data-driven"
([C48.1](01-the-cast.md)): the difficulty curve *is* the table rows.

> ✅ *Verified:* `rh("CopCountRecord")=0xFCAA46E2` appears **×22** and `rh("CopFormationRecord")=0xB5A53D76` **×22**
> in `GLOBAL/attributes.bin` — the per-Heat dispatch tables. Cop-type keys are present: `copsuv` ×16, `copheli`
> ×15, `copcross` ×7. Heat is named (`MinHeatLevel`, `SetWorldHeat`, `AnytimeEvents_HeatJump`).

## What Heat selects

At each level, Heat selects four things ([C48.3](03-escalation-ladder.md)):

1. **How many cops** (`CopCountRecord`) — the count `AICopManager` maintains
   ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).
2. **What mix and formation** (`CopFormationRecord`) — which cop *types* (`copsuv`, `copheli`, `copcross`,
   [above](#heat-is-a-scalar-that-indexes-tables)) and how they arrange around you.
3. **Which support strategy** is authorised — Leader → Heavy → AirSupport
   ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) — the reinforcement tier.
4. **How aggressive** the active goals may get — whether cops may only `AIGoalPursuit` or may escalate to
   `AIGoalPit`/`AIGoalRam` ([C48.3](03-escalation-ladder.md)).

So Heat is a *single dial* that, through the tables, controls the entire character of the chase — its size, its
composition, its reinforcements, and its aggression. This is elegant: one scalar, four consequences, all mediated
by data. Turn the dial up and every aspect of the pursuit intensifies together, coherently.

## The roster tiers

The cop *types* Heat selects (`CopFormationRecord`) form a **roster of tiers** — the verified cop shells:

- **Low Heat** — light sedans (`cop1`, `copmidsize`).
- **Mid Heat** — sportier and heavier cars (`copsport`, `copsuv` ×16, `copcross` ×7 — SUVs and cross-country
  units).
- **High Heat** — the heaviest ground units and the **helicopter** (`copheli` ×15).

Higher Heat *selects heavier rows* of these tiers — more units, and tougher ones. The reference counts
([above](#heat-is-a-scalar-that-indexes-tables)) hint at usage: `copsuv` (×16) and `copheli` (×15) are the
frequently-referenced mid/high-tier units. So the escalation you *feel* — from a lone cruiser to a swarm with a
chopper overhead — is the table walking up the roster tiers as Heat climbs.

> 🟡 *Reasoned:* the mapping of specific cop-type keys to Heat tiers (sedans low, SUVs mid, chopper high) is the
> pursuit design, consistent with the verified cop-type keys and the ×22 dispatch tables; the exact per-Heat rows
> are the vault table contents. The records, their counts, and the cop-type keys are verified.

## Heat as career standing

Heat is not only per-pursuit — it's also your **career standing** (the "Most Wanted" theme). The persistent Heat
associated with you (and with the Blacklist progression) determines the *baseline* intensity of pursuits you
attract:

- **A higher career Heat** means pursuits *start* heavier and escalate faster — the tables are indexed from a
  higher baseline.
- **`MinHeatLevel`** (verified) is the floor — pursuits can't drop below a Heat set by your standing.
- **Cooling down** ([C48.4](04-bust-evade.md)) reduces the *active* pursuit Heat, but your career standing persists.

So Heat bridges the moment-to-moment (this chase's intensity) and the long-term (your notoriety). The same scalar
that indexes the dispatch tables *this second* is the number that makes you Most Wanted *this save*. This dual role
is why Heat is the master variable — it's the pursuit's intensity *and* the game's progression, unified in one
number.

## RE implications

- **Heat is a scalar that indexes the dispatch tables** — `CopCountRecord` (×22) and `CopFormationRecord` (×22).
- **It selects** how many cops, what mix/formation, which support strategy, and how aggressive the goals may get.
- **The roster tiers** (sedans → SUVs → chopper) are walked up as Heat climbs — verified cop-type keys (`copsuv`,
  `copheli`, `copcross`).
- **Heat is also career standing** — `MinHeatLevel` floors it by your notoriety; one scalar, moment and long-term.

---

### Key takeaways

- **Heat** is the master **intensity scalar** — it **indexes the vault dispatch tables** `CopCountRecord` (×22) and
  `CopFormationRecord` (×22); higher Heat selects *heavier rows*.
- It selects four things: **how many** cops, **what mix/formation**, **which support** (Leader→Heavy→AirSupport),
  and **how aggressive** the goals.
- The **roster tiers** — sedans (`cop1`/`copmidsize`) → SUVs/cross (`copsuv` ×16, `copcross` ×7) → chopper
  (`copheli` ×15) — are climbed as Heat rises.
- One **dial, four consequences**, all mediated by data — which is why the pursuit is almost entirely
  data-driven.
- Heat is also **career standing** (`MinHeatLevel`) — the pursuit's intensity *and* the game's progression in one
  number.

**Continue:** [C48.3 — The escalation ladder](03-escalation-ladder.md) · [Chapter 48 hub](C48-Pursuit-Heat.md)
