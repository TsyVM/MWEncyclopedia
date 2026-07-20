# C46.1 — The Goal/Action Model

> **The one-sentence version:** an AI car's brain is a two-level system — a **goal** (sustained intent, e.g.
> "pursue") holds a fixed **menu of actions** (low-level behaviours, e.g. "race the line," "ram," "get unstuck"),
> and each tick the goal runs whichever action best fits the situation.

[← Chapter 46 hub](C46-AI-Goals-Actions.md) · [Next: C46.2 — The goal catalogue →](02-goal-catalog.md)

---

## Two levels: intent and behaviour

Most Wanted's AI separates *what a car wants* from *what it's doing right now* — two levels:

- **Goal** — the **sustained intent**. A goal is the car's current purpose: pursue the player, flee, be traffic,
  hold a roadblock. It changes slowly, on the timescale of the situation (a patrol cruiser spots you → its goal
  becomes pursuit).
- **Action** — the **low-level behaviour**. An action is a concrete driving behaviour the car executes *this
  tick*: race the racing line, ram the target, cut off-road, recover from being stuck. It changes fast, moment to
  moment.

A goal **holds a menu of actions** ([C46.5](05-action-menu.md)) — a fixed repertoire of the behaviours that intent
can call on — and each tick it **runs whichever action best fits** the current situation. So the goal is the
*strategy* and the actions are the *tactics*: the intent stays "pursue," while the behaviour switches between
racing the line, cutting off-road, and recovering, as conditions change.

> ✅ *Verified:* the goal and action classes are runtime classes in `speed.exe` — 15 `AIGoal*` and 17 `AIAction*`
> strings ([C46.2](02-goal-catalog.md), [C46.5](05-action-menu.md)); the goals register onto the list-head
> `0x0090D8E8`.

## The per-tick selection

The core loop of an AI car is the goal running its menu each tick:

```
each tick, for an AI car:
   goal = the car's current goal        (set by the manager, C47)
   action = goal.select_best(situation) // pick the best-fitting action from the menu
   action.run(car)                       // execute it → controls (throttle/brake/steer)
```

The `select_best` step is the AI's *judgement* — given the current situation (where's the target, am I stuck, am I
airborne, am I wrecked), which of my menu's actions is most appropriate? For a pursuit goal
([C46.2](02-goal-catalog.md)): normally race the line; if the line would lose the target, cut off-road; if stuck,
get unstuck; if wrecked, disengage. The action then produces the actual driving controls
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) — throttle, brake, steer — that the sim consumes
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).

So the AI is a two-level decision each tick: the goal (already chosen) selects an action, and the action drives.
The intelligence is split between *which goal* (the manager's job, [Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md))
and *which action* (the goal's job, per tick).

## The manager swaps goals

Goals don't swap themselves — a **manager** ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) decides
which goal a car has, and changes it as circumstances change:

- **A patrol cruiser** has `AIGoalPatrol` ([C46.2](02-goal-catalog.md)) — until it spots you speeding, when the
  manager (`AIPursuit`, [Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) swaps it to `AIGoalPursuit`.
- **As Heat rises** ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), the manager promotes the cop's goal to
  the aggressive ones — `AIGoalPit`, `AIGoalRam`, `AIGoalPullOver` — authorising contact.
- **When you escape**, the manager drops the goal back toward patrol/traffic.

So there are three timescales of AI decision: the **manager** changes the *goal* (slow, situational); the **goal**
selects the *action* (fast, per-tick); the **action** produces the *controls* (immediate). This layering is what
makes the AI both coherent (a stable intent) and responsive (moment-to-moment tactics), and it cleanly separates
the three concerns.

## Why a two-level design

Splitting AI into goals and actions is a classic, powerful architecture:

- **Reuse.** The same actions ([C46.5](05-action-menu.md)) — race, ram, get-unstuck — are reused across many goals'
  menus. A behaviour is written once and composed into whatever intents need it.
- **Composability.** A new intent is a new *menu* of existing actions ([C46.3](03-data-only-goals.md)) — no new
  behaviour code. This is why most goals are data-only ([C46.3](03-data-only-goals.md)).
- **Clear reasoning.** The AI is legible: read a goal's menu to know its repertoire, watch the per-tick selection
  to see its judgement. Debugging is "which goal, which action."
- **Tunable escalation.** The manager's goal-swapping ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) is
  data-driven thresholds — cop aggression is tuned by *when* goals swap, not by rewriting behaviour.

So the goal/action model is Most Wanted's answer to "how do you build varied, believable AI drivers": a small set
of reusable actions, composed into intents (goals), swapped by managers on tuned thresholds. The rest of the
chapter shows how *little* code this takes — because most goals are pure data
([C46.3](03-data-only-goals.md)).

## RE implications

- **AI is two levels** — goal (sustained intent) and action (per-tick behaviour); the goal holds a menu of actions.
- **Each tick** the goal selects the best-fitting action, which produces the driving controls.
- **A manager swaps goals** ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) as circumstances
  change — three timescales (manager → goal → action).
- **The two-level design** buys reuse (shared actions), composability (new intent = new menu), and tunable
  escalation.

---

### Key takeaways

- An AI car's brain is **two levels**: a **goal** (sustained intent) holding a **menu of actions** (low-level
  behaviours), running the best-fitting action each tick.
- **Three timescales**: the **manager** swaps the *goal* (slow), the **goal** selects the *action* (per-tick), the
  **action** produces the *controls* (immediate).
- The per-tick `select_best` is the AI's **judgement** — race the line, or cut off-road, or recover, per the
  situation.
- The design buys **reuse** (shared actions), **composability** (a new intent is a new menu), and **tunable
  escalation** (managers swap goals on data thresholds).
- This is verified: **15 `AIGoal*`** and **17 `AIAction*`** classes, on the goal list-head `0x0090D8E8`.

**Continue:** [C46.2 — The goal catalogue](02-goal-catalog.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md)
