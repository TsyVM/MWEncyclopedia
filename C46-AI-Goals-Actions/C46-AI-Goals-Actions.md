# Chapter 46 — AI Architecture: Goals & Actions

> **Goal of this chapter:** decode the mind that drives every non-player car — the **goal/action** architecture: a
> goal is a sustained intent holding a menu of actions, and ten of the ~15 goals are *data-only* (sharing one
> 12-method base vtable `0x00892B20`), differing only in their action menu, while four carry large override
> vtables where real behaviour is coded — all verified against `speed.exe`.

Every cop, racer, and traffic car is driven by an **AI** ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) whose
brain is a **goal/action** system. This chapter decodes that architecture: what a goal is, what an action is, how a
goal selects actions each tick, and the striking structural fact that most goals are *pure data* — the same code
(one shared vtable) specialised only by which actions each goal's constructor installs. It's one of the most
elegant and most *verified* systems in the engine.

> **Verified against the executable.** The AI goals are runtime classes in `speed.exe`: **15 `AIGoal*`** strings
> (`AIGoalPursuit`, `AIGoalFleePursuit`, `AIGoalPatrol`, `AIGoalRacer`, `AIGoalTraffic`, `AIGoalRam`, `AIGoalPit`,
> `AIGoalPullOver`, `AIGoalStopShort`, `AIGoalHeadOnRam`, `AIGoalStaticRoadBlock`, `AIGoalHeliPursuit`,
> `AIGoalHeliExit`, `AIGoalHeliRoadBlock`, `AIGoalNone`) and **17 `AIAction*`** strings. The **shared base vtable
> `0x00892B20` has exactly 12 methods** (verified by pointer count) — the data-only goals. Four override goals
> carry large vtables, all verified: `AIGoalFleePursuit` `0x00892D00`/**94**, `AIGoalHeliPursuit`
> `0x00892D10`/**90**, `AIGoalHeliExit` `0x00892D20`/**86**, `AIGoalRacer` `0x00892D30`/**82**. Goals register onto
> the runtime list-head `0x0090D8E8`. The tactical goals are vault-tuned: `AIGoalPit` ×9, `AIGoalStaticRoadBlock`
> ×11 in `attributes.bin`.

---

## Deep-dive pages

- [C46.1 — The goal/action model](01-goals-and-actions.md): intent (goal) selecting behaviour (action) each tick.
- [C46.2 — The goal catalogue](02-goal-catalog.md): the 15 goals and what each intends.
- [C46.3 — The data-only goals](03-data-only-goals.md): ten goals, one 12-method vtable, different menus.
- [C46.4 — The override goals](04-override-goals.md): Racer (82), FleePursuit (94), HeliPursuit (90), HeliExit
  (86).
- [C46.5 — The action menu](05-action-menu.md): the 17 actions and how a goal runs the best-fitting one.
- [C46.6 — Reading AI in RE](06-reading-ai.md): navigating the goal/action system.

---

## 46.1 Goals and actions

A **goal** is a car's *sustained intent* — "chase the player," "flee," "be traffic," "hold a roadblock"
([C46.1](01-goals-and-actions.md)). It holds a fixed **menu of actions**, and each tick runs whichever action best
fits the situation. An **action** is a *low-level behaviour* — "race the line," "ram," "get unstuck," "recover from
airborne." So the split is intent (goal, slow-changing) vs. behaviour (action, per-tick). A **manager**
([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) swaps which goal is active as circumstances change
(patrol → pursuit as Heat rises).

## 46.2 The goal catalogue

The 15 goals ([C46.2](02-goal-catalog.md)) cover every AI role: **cop** (`AIGoalPursuit`, `AIGoalPit`, `AIGoalRam`,
`AIGoalPullOver`, `AIGoalHeadOnRam`, `AIGoalStopShort`, `AIGoalStaticRoadBlock`, `AIGoalPatrol`), **helicopter**
(`AIGoalHeliPursuit`, `AIGoalHeliExit`, `AIGoalHeliRoadBlock`), **racer/suspect** (`AIGoalRacer`,
`AIGoalFleePursuit`), **traffic** (`AIGoalTraffic`), and the inert `AIGoalNone`. The escalation of a pursuit
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) is a walk up the aggressive cop goals.

## 46.3 The data-only goals

The standout structural fact ([C46.3](03-data-only-goals.md)): **ten of the goals share one 12-method base vtable
`0x00892B20`** — they are *data-only* specialisations, differing only in the **action menu** their constructor
installs. `AIGoalPursuit`, `AIGoalPit`, `AIGoalRam`, `AIGoalPullOver`, `AIGoalPatrol`, `AIGoalTraffic`, etc. run the
*same code*; the behavioural difference between "chase," "PIT," and "ram" is *which actions are in the menu* plus
the vault thresholds that swap them. This is data-driven AI at its purest.

## 46.4 The override goals

Four goals carry their **own large vtables** ([C46.4](04-override-goals.md)) — where real behaviour is coded:
`AIGoalRacer` (**82 methods**, the racing brain — line, drafting, catch-up), `AIGoalFleePursuit` (**94** — the
most-overridden, because evading intelligently is hard), `AIGoalHeliPursuit` (**90** — airborne tracking,
spotlight), `AIGoalHeliExit` (**86**). These are the AIs with genuine, complex intelligence, correspondingly harder
to modify without disassembly.

## 46.5 The action menu

Each goal's **menu** is a subset of the 17 actions ([C46.5](05-action-menu.md)) — e.g. `AIGoalPursuit`'s menu is
`Race · PursuitOffRoad · Airborne · GetUnstuck · TooDamaged · Traffic`. Each tick, the goal runs the action that
best fits: race the line normally, cut off-road (`PursuitOffRoad`) when the line would lose you, recover
(`GetUnstuck`/`Airborne`) when stuck or launched, disengage (`TooDamaged`) when wrecked. The menu is the goal's
*repertoire*; the per-tick selection is its *judgement*.

---

### Key takeaways

- AI is a **goal/action** system: a **goal** is sustained intent holding a **menu of actions**; each tick it runs
  the best-fitting action; a **manager** swaps goals as circumstances change.
- There are **15 goals** (verified `AIGoal*` strings) and **17 actions** (`AIAction*`) — covering cop, helicopter,
  racer/suspect, and traffic roles.
- **Ten goals are data-only** — sharing one **12-method base vtable `0x00892B20`** (verified), differing only in
  their **action menu** and vault thresholds.
- **Four goals override** with large vtables (verified): `FleePursuit` (94), `HeliPursuit` (90), `HeliExit` (86),
  `Racer` (82) — where real AI behaviour is coded.
- The tactical goals are **vault-tuned** (`AIGoalPit` ×9, `AIGoalStaticRoadBlock` ×11) — cop aggression is data.

**Next:** [Chapter 47 — AI Driver Brain & Vehicle Hierarchy](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md): the
managers and AI vehicle classes that host these goals.
