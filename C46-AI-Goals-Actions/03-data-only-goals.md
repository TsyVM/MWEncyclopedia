# C46.3 — The Data-Only Goals

> **The one-sentence version:** ten of the goals share **one 12-method base vtable `0x00892B20`** — they run the
> *same code* and differ only in the **action menu** their constructor installs, so "chase," "PIT," "ram," and
> "pull over" are the same class with different menus plus vault thresholds.

[← C46.2 — The goal catalogue](02-goal-catalog.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md) ·
[Next: C46.4 — The override goals →](04-override-goals.md)

---

## One vtable, ten goals

The most striking structural fact of the AI is that **most goals are pure data**. Ten of the ~15 goals
([C46.2](02-goal-catalog.md)) — `AIGoalPursuit`, `AIGoalPit`, `AIGoalRam`, `AIGoalPullOver`, `AIGoalHeadOnRam`,
`AIGoalStopShort`, `AIGoalPatrol`, `AIGoalTraffic`, `AIGoalStaticRoadBlock`, `AIGoalNone` — **share a single base
vtable at `0x00892B20`**, which has exactly **12 methods** (verified by pointer count).

Sharing a vtable means sharing *all the code*: these ten goals have **identical behaviour** at the code level. They
are not ten different implementations — they are ten *instances* of the same 12-method class, distinguished only by
their **data**: the action menu each installs ([C46.5](05-action-menu.md)) and its vault thresholds
([C46.2](02-goal-catalog.md)).

> ✅ *Verified:* the shared base vtable at `0x00892B20` has exactly **12 methods** (12 consecutive `.text` pointers,
> confirmed by count). The data-only goals register on the goal list-head `0x0090D8E8` and differ by their
> constructor-installed action menu.

## The difference is the menu

If ten goals share the same code, what makes `AIGoalPursuit` (chase) different from `AIGoalRam` (shunt)? Their
**action menus** ([C46.5](05-action-menu.md)) — the set of actions each goal's constructor installs:

| Goal | Action menu (the difference) |
|---|---|
| `AIGoalPursuit` | Race · PursuitOffRoad · Airborne · GetUnstuck · TooDamaged · Traffic |
| `AIGoalPit` | Airborne · GetUnstuck · PursuitOffRoad · Race · **Ram** · TooDamaged |
| `AIGoalRam` | (same aggressive set, `…Ram…`) — sustained shunting |
| `AIGoalPullOver` | (same aggressive set, `…Ram…`) — box and stop |
| `AIGoalHeadOnRam` | Airborne · GetUnstuck · **HeadOnRam** · Race · TooDamaged |
| `AIGoalStopShort` | Airborne · GetUnstuck · **StopShort** · TooDamaged |
| `AIGoalPatrol` | **Traffic** · TooDamaged |
| `AIGoalTraffic` | **Traffic** only |
| `AIGoalStaticRoadBlock` | **StaticRoadBlock** only |
| `AIGoalNone` | **None** |

The 12-method base code is the same "run the best-fitting action from my menu each tick"
([C46.1](01-goals-and-actions.md)) — but *which* actions are in the menu changes everything. `AIGoalPursuit` can
race and cut off-road but *can't ram* (no Ram in its menu); `AIGoalPit` adds Ram, so it *can* make contact. The
behavioural difference between chasing and PITting is one action in a menu.

> 🟡 *Reasoned:* the specific action menus per goal are the recovered goal→action structure; the *mechanism* (menu
> installed by the constructor, run by the shared 12-method base) is verified via the shared vtable. The exact menu
> contents are from the recovered map and consistent with observed cop behaviour.

## Why this is brilliant

The data-only-goals design is a masterclass in data-driven AI:

- **New intent, no code.** Want a new cop behaviour? Define a new goal = a new *menu* of existing actions. No new
  class, no new vtable — just data ([C46.1](01-goals-and-actions.md)). This is why there are ten goals for the cost
  of one class.
- **Behaviour is legible and tunable.** A goal's behaviour is *readable* as its menu — you can see exactly what
  `AIGoalPit` can do by listing its actions. And cop aggression is tuned by the manager's swap *thresholds*
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), not by editing behaviour.
- **The actions carry the real code.** Because the goals are thin, the *actions* ([C46.5](05-action-menu.md)) are
  where the driving behaviour lives — Ram knows how to ram, Race knows how to race. Reuse those across menus and
  you compose all the intents.
- **Consistency.** All ten goals *select* actions the same way (the shared 12 methods), so the AI's decision logic
  is uniform — one place to get right, ten goals to benefit.

So the ten data-only goals are the ultimate expression of composition ([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)):
the cop's whole repertoire of intents is *one class* configured ten ways by menus and thresholds. The "code" of
cop AI is mostly the shared 12 methods plus the actions; the "content" is the menus and the vault. This is why cop
behaviour can be tuned so extensively through data — most of it *is* data.

## The four exceptions

Four goals *don't* share the base vtable — `AIGoalRacer`, `AIGoalFleePursuit`, `AIGoalHeliPursuit`, `AIGoalHeliExit`
carry their own large override vtables ([C46.4](04-override-goals.md)). These are the goals whose behaviour is *too
complex* to express as a menu of shared actions — racing intelligence, intelligent evasion, and helicopter control
need real, overridden code. So the split is clean: the ten *tactical-but-simple* cop/traffic goals are data-only;
the four *genuinely-intelligent* goals override. Recognising which is which ([C46.4](04-override-goals.md)) tells
you where the AI's real code complexity lives.

## RE implications

- **Ten goals share one 12-method base vtable `0x00892B20`** — identical code, differing only by data (verified).
- **The difference is the action menu** — `AIGoalPursuit` (no Ram) vs. `AIGoalPit` (adds Ram) is one menu entry.
- **New intent = new menu** — no new code; the actions carry the real behaviour.
- **Four goals override** ([C46.4](04-override-goals.md)) — the genuinely complex ones (Racer, Flee, Heli×2).

---

### Key takeaways

- **Ten goals share a single 12-method base vtable `0x00892B20`** (verified) — they run **identical code**,
  differing only in the **action menu** their constructor installs.
- The behavioural difference between chase, PIT, ram, and pull-over is **which actions are in the menu** (e.g.
  `AIGoalPit` adds `Ram` to `AIGoalPursuit`'s set) plus vault thresholds.
- This is **data-driven AI at its purest** — a new intent is a new menu, not new code; ten goals for the cost of
  one class.
- The **actions carry the real driving code** ([C46.5](05-action-menu.md)); the goals are thin selectors, all
  deciding uniformly (the shared 12 methods).
- **Four goals override** with their own vtables ([C46.4](04-override-goals.md)) — the genuinely intelligent ones,
  where the AI's real code complexity lives.

**Continue:** [C46.4 — The override goals](04-override-goals.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md)
