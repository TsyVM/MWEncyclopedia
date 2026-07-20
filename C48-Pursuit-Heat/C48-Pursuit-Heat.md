# Chapter 48 — Pursuit & Heat: the State Machine

> **Goal of this chapter:** decode Most Wanted's signature system — the pursuit — as `AIPursuit` (98 methods)
> running a chase's state machine: **Heat** (the intensity scalar that indexes the dispatch tables), the
> **escalation ladder** (goal-swaps and reinforcements as Heat climbs), and the **bust/evade** resolution — almost
> entirely data-driven on a small set of verified code classes.

The pursuit is Most Wanted's beating heart, and — unlike most of the engine — it is **almost entirely data-driven
on top of a small set of code classes**. This chapter decodes the *chase itself*: how `AIPursuit`
([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)) starts a pursuit, drives it up the Heat ladder (adding
cars, helicopters, and roadblocks), and resolves it in a bust or an escape. The *fleet supply* (how cops spawn and
form up) is the next chapter ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)); here
is the *state machine* that runs the drama.

> **Verified against the executable and vault.** `AIPursuit` (the chase director) is a verified vtable at
> `0x00892770` (**98 methods**, [Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)). The pursuit lifecycle is
> named in `speed.exe`: `PursuitBegins`, `PursuitStart`, `PursuitApproaching`, `PursuitEscalation`,
> `PursuitAddsCar`, `PursuitAddsHeli`, `PursuitAddsRoadblock`, `PursuitBreaker`, `PursuitEnds`, `PursuitOver`,
> `Busted`. Heat is named: `AnytimeEvents_HeatJump`, `MinHeatLevel`, `SetWorldHeat`, `Hud_HeatMeter`. The **dispatch
> tables are heavily-used vault records**: `rh("CopCountRecord")=0xFCAA46E2` **×22** and
> `rh("CopFormationRecord")=0xB5A53D76` **×22** in `attributes.bin`, with cop-type keys `copsuv` ×16, `copheli`
> ×15, `copcross` ×7. The `pursuit` vault collection (`0xDAA252C2`) holds the tuning.

---

## Deep-dive pages

- [C48.1 — The cast & the AIPursuit director](01-the-cast.md): who runs a pursuit.
- [C48.2 — Heat: the escalation variable](02-heat.md): the scalar that indexes the dispatch tables.
- [C48.3 — The escalation ladder](03-escalation-ladder.md): goal-swaps and reinforcements up the rungs.
- [C48.4 — The bust & the evade](04-bust-evade.md): how a pursuit ends — caught or escaped.
- [C48.5 — Reading pursuit in RE](05-reading-pursuit.md): navigating the pursuit system.

---

## 48.1 The cast

A pursuit is run by a small cast ([C48.1](01-the-cast.md)): **`AIPursuit`** is the *director* of one chase (Heat,
timers, escalation, which goal each cop holds, [Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md));
**`AICopManager`** is the *fleet* (spawns units against the Heat tables,
[Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)); **`AIRoadBlock`** sites roadblocks;
**`AIVehicleCopCar`** (324 methods) is the cop brain ([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md));
the helicopter, spike strips, and the AI→audio bridge round out the players. `AIPursuit` conducts them all.

## 48.2 Heat

**Heat** ([C48.2](02-heat.md)) is the master **intensity scalar** of a pursuit (and your career standing). It
*indexes the vault dispatch tables* — `CopCountRecord` (×22) and `CopFormationRecord` (×22) — so at each Heat level
the game knows **how many** cops to keep on you, **what mix and formation** (sedans → SUVs → the chopper), **which
support strategy** is authorised, and **how aggressive** the goals may get. Higher Heat simply selects heavier rows
of these tables. `AnytimeEvents_HeatJump` fires the escalation beats.

## 48.3 The escalation ladder

As Heat climbs, `AIPursuit` walks the **escalation ladder** ([C48.3](03-escalation-ladder.md)) — promoting cops up
the goal set ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)) from `AIGoalPursuit` (chase) to
`AIGoalPit`/`AIGoalRam`/`AIGoalPullOver` (make contact), and calling in reinforcements: the verified events
`PursuitAddsCar`, `PursuitAddsHeli`, and `PursuitAddsRoadblock` are the rungs. The *menus are code*
([Chapter 46](../C46-AI-Goals-Actions/03-data-only-goals.md)); the *thresholds are data* (the `pursuit` vault) — so
cop aggression is tuned by *when* goals swap, not by rewriting behaviour.

## 48.4 The bust & the evade

A pursuit ends one of two ways ([C48.4](04-bust-evade.md)): **bust** (caught — `Busted`) or **evade** (escaped —
`PursuitEnds`/`PursuitOver`). The bust is governed by a **bust envelope** — proximity radii and a speed gate
(`BustSpeed`): get boxed in, slow/stopped, and held close for long enough, and the busted meter fills. Evasion is
the inverse — break line-of-sight and open distance ([Chapter 46](../C46-AI-Goals-Actions/04-override-goals.md), the
`FleePursuit` brain) until the cool-down timer expires. `PursuitBreaker` set-pieces (droppable environment) and
spike strips are the world tools that tip the balance.

---

### Key takeaways

- The pursuit is **almost entirely data-driven on a small set of code classes** — `AIPursuit` (98 methods) is the
  chase director/state machine.
- **Heat** is the intensity scalar — it **indexes the dispatch tables** `CopCountRecord` (×22) and
  `CopFormationRecord` (×22): how many cops, what mix/formation, which support, how aggressive.
- The **escalation ladder** promotes cop goals (Pursuit → Pit/Ram/PullOver) and adds reinforcements
  (`PursuitAddsCar`/`AddsHeli`/`AddsRoadblock`) — **menus are code, thresholds are data**.
- A pursuit ends in a **bust** (`Busted` — boxed, slow, held close) or an **evade** (`PursuitEnds`/`Over` — break
  contact, open distance, cool down).
- The whole lifecycle is **named in the executable** (`PursuitBegins` → escalation → `Busted`/`PursuitOver`) —
  recoverable and tunable.

**Next:** [Chapter 49 — Cops: Dispatch, Formations, Roadblocks & Bust](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md):
the fleet that supplies the pursuit.
