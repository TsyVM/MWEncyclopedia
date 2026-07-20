# C46.5 — The Action Menu

> **The one-sentence version:** the 17 verified `AIAction*` classes are the AI's low-level behaviours — Race,
> Traffic, Ram, HeadOnRam, Strafe, StopShort, PursuitOffRoad, Airborne, GetUnstuck, JackKnife, TooDamaged,
> StaticRoadBlock, Spline, Heli×2, None — and each goal's menu is a subset the goal runs the best-fitting one from
> each tick.

[← C46.4 — The override goals](04-override-goals.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md) ·
[Next: C46.6 — Reading AI in RE →](06-reading-ai.md)

---

## The 17 actions

Where goals are *intents* ([C46.2](02-goal-catalog.md)), **actions are behaviours** — the concrete things an AI car
does. The 17 verified `AIAction*` classes:

| Action | Behaviour |
|---|---|
| `AIActionRace` | drive the racing line at speed (the workhorse) |
| `AIActionTraffic` | follow the lane as civilian traffic |
| `AIActionRam` | ram the target — make contact to damage/shove |
| `AIActionHeadOnRam` | intercept and collide head-on |
| `AIActionStrafe` | weave/strafe alongside the target |
| `AIActionStopShort` | pull ahead and brake to stop the target short |
| `AIActionPursuitOffRoad` | cut off-road when the road line would lose the target |
| `AIActionAirborne` | control the car while airborne (jumps) |
| `AIActionGetUnstuck` | recover when stuck against geometry |
| `AIActionJackKnife` | a jackknife manoeuvre (block/spin) |
| `AIActionTooDamaged` | disengage when too wrecked to continue |
| `AIActionStaticRoadBlock` | hold position in a roadblock |
| `AIActionSpline` | follow a scripted spline path (on-rails) |
| `AIActionHeliPursuit` | the helicopter's airborne chase behaviour |
| `AIActionHeliExit` | the helicopter's departure behaviour |
| `AIActionRace`, … | (plus `AIAction` base and `AIActionNone`) |

> ✅ *Verified:* all 17 `AIAction*` strings are present in `speed.exe`, including the base `AIAction` and
> `AIActionNone`. They are the behaviours the goals' menus draw from ([C46.3](03-data-only-goals.md)).

## Actions are the reusable behaviours

The key role of actions is **reuse** ([C46.1](01-goals-and-actions.md)): a single action is written once and appears
in *many* goals' menus. `AIActionRace` (drive the line) is in `AIGoalPursuit`, `AIGoalPit`, `AIGoalRam`,
`AIGoalPullOver` ([C46.3](03-data-only-goals.md)) — because all those cop intents involve *chasing*, which is
racing the line. Likewise:

- **`AIActionGetUnstuck` and `AIActionAirborne`** appear in nearly every driving goal — any car can get stuck or
  launched, so these recovery behaviours are shared everywhere.
- **`AIActionTooDamaged`** is in most goals — any AI can be wrecked and need to disengage.
- **`AIActionTraffic`** is in `AIGoalTraffic` (only) and `AIGoalPatrol` (which drives as traffic until provoked,
  [C46.2](02-goal-catalog.md)).

So the actions are the *vocabulary* of AI behaviour, and the goals are *sentences* composed from that vocabulary
([C46.3](03-data-only-goals.md)). The real driving code lives in the actions (Ram knows how to ram); the goals just
select among them. This is why the data-only goals ([C46.3](03-data-only-goals.md)) work — all the behaviour they
need already exists as reusable actions.

## Best-fitting selection

Each tick, the goal runs the **best-fitting** action from its menu ([C46.1](01-goals-and-actions.md)) — a
priority/condition selection. For `AIGoalPursuit` (menu: `Race · PursuitOffRoad · Airborne · GetUnstuck ·
TooDamaged · Traffic`):

```
each tick:
   if too damaged        → AIActionTooDamaged    (disengage)
   elif airborne         → AIActionAirborne      (control the jump)
   elif stuck            → AIActionGetUnstuck     (recover)
   elif line would lose  → AIActionPursuitOffRoad (cut across)
   elif lost sight       → AIActionTraffic        (blend in, C48)
   else                  → AIActionRace           (chase the line)
```

The selection is by *situation* — the exceptional conditions (damaged, airborne, stuck) take priority, then the
tactical ones (off-road cut, blend in), with the default (`Race`) when nothing special applies. So the AI's
moment-to-moment intelligence is this per-tick triage: handle the emergency, else pursue the tactic, else do the
default. Watching an AI car, you *see* this — a chasing cop that hits a jump switches to `Airborne`, lands, and
resumes `Race`.

> 🟡 *Reasoned:* the priority ordering of the selection (emergencies first, default last) is the natural reading of
> the action menus and observed AI behaviour; the exact per-action fitness conditions are in the goal's selection
> code (the shared 12 methods, [C46.3](03-data-only-goals.md)). The action set and the menus are verified.

## Actions produce controls

At the bottom, an action produces the car's **controls** — throttle, brake, steer
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) — which the sim consumes
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)). This is where the AI meets the physics:

- **`AIActionRace`** computes the steer/throttle to follow the racing line at the right speed
  ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)).
- **`AIActionRam`** steers *at* the target and applies throttle to make contact
  ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)).
- **`AIActionGetUnstuck`** applies reverse/steer to free the car from geometry.

So the action is the final translator from AI decision to physical input — the AI's version of the player's hands
on the controls ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)). The whole goal/action stack exists to answer
one question each tick — *what throttle, brake, and steer?* — and the selected action answers it. From intent
(goal) to behaviour (action) to controls (throttle/brake/steer) to motion (the sim) is the full path from AI mind
to moving car.

## RE implications

- **17 actions** are the AI's reusable behaviours — Race, Ram, GetUnstuck, PursuitOffRoad, TooDamaged, etc.
- **Actions are reused across goals' menus** — `Race` in every chase goal, `GetUnstuck`/`Airborne` almost
  everywhere.
- **Best-fitting selection** — each tick the goal runs the highest-priority applicable action (emergencies first,
  default last).
- **Actions produce controls** — throttle/brake/steer — the AI's hands on the car, consumed by the sim.

---

### Key takeaways

- The **17 verified actions** are the AI's low-level behaviours (Race, Traffic, Ram, HeadOnRam, Strafe, StopShort,
  PursuitOffRoad, Airborne, GetUnstuck, JackKnife, TooDamaged, StaticRoadBlock, Spline, Heli×2, None).
- Actions are the **reusable vocabulary** — one action appears in many goals' menus (`Race` in every chase goal),
  which is why data-only goals ([C46.3](03-data-only-goals.md)) work.
- Each tick the goal runs the **best-fitting** action by **priority/situation** — emergencies (damaged, airborne,
  stuck) first, tactics next, default (`Race`) last.
- Actions **produce the car's controls** (throttle/brake/steer) — the AI's hands, consumed by the sim
  ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).
- The full path is **intent (goal) → behaviour (action) → controls → motion** — the AI mind driving the physical
  car.

**Continue:** [C46.6 — Reading AI in RE](06-reading-ai.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md)
