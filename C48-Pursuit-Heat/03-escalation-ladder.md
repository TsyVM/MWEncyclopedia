# C48.3 — The Escalation Ladder

> **The one-sentence version:** as Heat climbs, `AIPursuit` walks an escalation ladder — promoting cop goals from
> chase (`AIGoalPursuit`) to contact (`AIGoalPit`/`Ram`/`PullOver`) and calling reinforcements (`PursuitAddsCar`,
> `PursuitAddsHeli`, `PursuitAddsRoadblock`) — where the goal menus are code and the thresholds are data.

[← C48.2 — Heat](02-heat.md) · [Chapter 48 hub](C48-Pursuit-Heat.md) ·
[Next: C48.4 — The bust & the evade →](04-bust-evade.md)

---

## Two axes of escalation

As a pursuit intensifies ([C48.2](02-heat.md)), `AIPursuit` ([C48.1](01-the-cast.md)) escalates along **two axes**:

- **Aggression** — the cops' *goals* ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)) get more
  aggressive: from just chasing (`AIGoalPursuit`) to making contact (`AIGoalPit`, `AIGoalRam`, `AIGoalPullOver`,
  `AIGoalHeadOnRam`).
- **Force** — *more and heavier units* join: additional cars (`PursuitAddsCar`), the helicopter
  (`PursuitAddsHeli`), and roadblocks (`PursuitAddsRoadblock`), from the dispatch tables ([C48.2](02-heat.md)).

Both climb with Heat. Early, one or two cruisers chase (`AIGoalPursuit`); later, a swarm rams and boxes you
(`AIGoalPit`/`Ram`) while a chopper tracks overhead and roadblocks wait ahead. The escalation you experience is
`AIPursuit` walking both axes up together as Heat rises.

> ✅ *Verified:* the reinforcement events are named in `speed.exe` — `PursuitAddsCar`, `PursuitAddsHeli`,
> `PursuitAddsRoadblock`, and `PursuitEscalation`; `AnytimeEvents_HeatJump` fires a Heat increase. The aggressive
> goals (`AIGoalPit` ×9, `AIGoalStaticRoadBlock` ×11) are vault-tuned ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)).

## The goal-swap ladder

The aggression axis is a **ladder of goal-swaps** ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)):
`AIPursuit` promotes each assigned cop's goal up the rungs as Heat authorises:

```
AIGoalPursuit      chase — race the line after you (no contact)
   ↓ (Heat rises)
AIGoalPit          PIT — make contact to spin you out
AIGoalRam          sustained ramming
AIGoalPullOver     box and stop you
AIGoalHeadOnRam    intercept head-on
AIGoalStopShort    pull ahead and brake-check
```

Each rung *adds authorisation to make contact* ([C46.3](../C46-AI-Goals-Actions/03-data-only-goals.md)) — recall
that `AIGoalPit`'s menu adds `Ram` to `AIGoalPursuit`'s set, so a promoted cop *can now ram*. `AIPursuit` decides
*when* to promote (the escalation schedule, [C48.1](01-the-cast.md)) based on Heat and the pursuit's progress. So
climbing the ladder is `AIPursuit` swapping cops' goals to more-aggressive ones — turning a chase into a battle.

## Menus are code, thresholds are data

The escalation embodies the pursuit's core design principle ([C48.2](02-heat.md)):

- **The goal menus are code.** What `AIGoalPit` *can do* (its action menu, [C46.3](../C46-AI-Goals-Actions/03-data-only-goals.md))
  is fixed in the shared goal code. Ramming behaviour is `AIActionRam`, written once.
- **The swap thresholds are data.** *When* `AIPursuit` promotes a cop from `Pursuit` to `Pit` — the Heat level, the
  timing — is **vault data** (the `pursuit` collection, [C48.2](02-heat.md)).

So you tune cop aggression by editing *thresholds* (make cops PIT sooner, at lower Heat), not by rewriting
*behaviour*. This is why the pursuit is so moddable ([C48.5](05-reading-pursuit.md)): the *drama* — how quickly and
how hard cops escalate — is entirely in the data, over a fixed set of coded behaviours. A designer dials the whole
difficulty of the cop war through the escalation thresholds, without touching a line of goal or action code.

> 🟡 *Reasoned:* the specific rung order and per-Heat thresholds are the pursuit design/vault contents; the
> *mechanism* (goal-swaps up a ladder, menus in code, thresholds in the `pursuit` vault) is grounded in the
> verified goals ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) and the reinforcement events. The
> events and vault-tuned goals are verified.

## Reinforcements: cars, air, roadblocks

The force axis adds units via three verified events ([C48.2](02-heat.md)):

- **`PursuitAddsCar`** — more cruisers join, up to the `CopCountRecord` cap for the Heat
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)). The chase grows.
- **`PursuitAddsHeli`** — the **helicopter** (`copheli`, [C48.2](02-heat.md)) is dispatched — airborne tracking
  ([C46.4](../C46-AI-Goals-Actions/04-override-goals.md)) that you can't shake by breaking ground line-of-sight
  alone.
- **`PursuitAddsRoadblock`** — `AIRoadBlock` ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
  sites a roadblock ahead of you — a set-piece to stop or slow you.

Each is a distinct escalation beat, fired by `AIPursuit` as Heat authorises. Together with the goal-swaps, they
make a high-Heat pursuit qualitatively different from a low one: not just *more* of the same, but *new kinds* of
threat — contact tactics, air support, and roadblocks appearing as you climb. This layered escalation is what makes
Most Wanted's pursuits build so memorably from a single cruiser to a city-wide manhunt.

## RE implications

- **Escalation has two axes** — aggression (cop goals: Pursuit → Pit/Ram/PullOver) and force (reinforcements:
  cars/heli/roadblocks).
- **The goal-swap ladder** — `AIPursuit` promotes each cop's goal up the rungs as Heat authorises contact.
- **Menus are code, thresholds are data** — tune aggression by *when* goals swap (vault), not behaviour.
- **Reinforcements** — `PursuitAddsCar`/`AddsHeli`/`AddsRoadblock` — add new *kinds* of threat, not just more
  units.

---

### Key takeaways

- As Heat climbs, `AIPursuit` escalates along **two axes**: **aggression** (cop goals) and **force**
  (reinforcements).
- The **goal-swap ladder** promotes each cop from `AIGoalPursuit` (chase) up to `AIGoalPit`/`Ram`/`PullOver`
  (contact) — each rung authorises ramming/boxing.
- **Menus are code, thresholds are data** — cop aggression is tuned by *when* goals swap (the `pursuit` vault), not
  by rewriting behaviour — the pursuit's moddability.
- **Reinforcements** are verified events — `PursuitAddsCar`, `PursuitAddsHeli`, `PursuitAddsRoadblock` — adding new
  *kinds* of threat (contact, air, roadblocks).
- Layered escalation makes a high-Heat pursuit **qualitatively** different — a single cruiser becomes a city-wide
  manhunt.

**Continue:** [C48.4 — The bust & the evade](04-bust-evade.md) · [Chapter 48 hub](C48-Pursuit-Heat.md)
