# C47.3 — The Managers

> **The one-sentence version:** above the driver brains sit the managers — `AIPursuit` (98 methods, the chase
> director owning a pursuit's state), `AICopManager` (43, the fleet manager owning all cops), `AITrafficManager`
> (29, the ambient population), and `AIRoadBlock` (62, roadblocks) — coordinators on list-head `0x0092C668`.

[← C47.2 — The player is an AI](02-player-is-ai.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md) ·
[Next: C47.4 — Navigation & the named systems →](04-navigation-systems.md)

---

## Coordinators above the brains

A driver brain ([C47.1](01-aivehicle-hierarchy.md)) drives *one* car. But the AI needs coordination *across* cars —
who spawns, who's assigned to a chase, what intent each holds. That's the **managers**: singletons that "decide who
exists, what they intend, and how the world is wired," registered on the manager/activity list-head `0x0092C668`.
All verified vtables:

| Manager | Hash | vtable | Methods | Owns |
|---|---|---|---|---|
| `AIPursuit` | `0x1F319B62` | `0x00892770` | **98** | the state of one pursuit |
| `AICopManager` | `0x5DB210B6` | `0x008918D8` | 43 | all the cops (the fleet) |
| `AIRoadBlock` | `0xD34FE5EF` | `0x00892130` | 62 | roadblock orchestration |
| `AITrafficManager` | `0xC91D5DF7` | `0x00890F18` | 29 | the ambient traffic population |

> ✅ *Verified:* all four manager vtables are confirmed by pointer count — `AIPursuit` `0x00892770`/98,
> `AICopManager` `0x008918D8`/43, `AIRoadBlock` `0x00892130`/62, `AITrafficManager` `0x00890F18`/29. They register
> on the manager/activity list `0x0092C668`.

## AIPursuit: the chase director

**`AIPursuit`** (98 methods, 652 B — one of the heaviest non-entity classes) is the **brain of a chase**. One
`AIPursuit` owns *the state of a pursuit*:

- **The roster** — which cops are assigned to this chase.
- **The Heat level** — the pursuit's intensity ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **The timers** — bust/evade timers that decide when you're caught or escaped.
- **The escalation schedule** — when to promote cops from `AIGoalPursuit` to `AIGoalPit`/`AIGoalRam`
  ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)) as Heat climbs.

So `AIPursuit` is effectively a **small state machine plus rosters** — it's what *runs* a pursuit
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)): promoting cop goals as Heat rises, calling for roadblocks
([C47.3](#airoadblock-the-roadblock-orchestrator)) and air support, and firing the pursuit event beats. It talks to
`AICopManager`, `AIRoadBlock`, `AIVehicleCopCar`, and the audio bridge. To understand a chase, `AIPursuit` is the
class — it's the subject of the next chapter ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).

## AICopManager: the fleet manager

Where `AIPursuit` owns *one chase*, **`AICopManager`** (43 methods) owns *all the cops*:

- **The roster of active units** — every cop currently in the world.
- **Spawning/despawning** against the Heat tables (`CopCountRecord`/`CopFormationRecord`,
  [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) for the current Heat.
- **Dispatch of reinforcements** — via the Leader/Heavy/AirSupport strategies
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).

Its per-frame update (`OnTask`) is a verified pipeline of sub-updates
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)): update patrols, update pursuits,
update roadblocks, update spawn requests — the fleet's heartbeat. `AICopManager` is the *supply* side of the cop
war (how many cops, spawned where), while `AIPursuit` is the *tactics* side (what the assigned cops do). Together
they are "how cops work" — the detail is in [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md).

## AIRoadBlock: the roadblock orchestrator

**`AIRoadBlock`** (62 methods, 168 B) orchestrates **roadblocks**. When `AIPursuit` calls for one, `AIRoadBlock`:

- **Chooses a site** ahead of you on the road network ([C47.4](04-navigation-systems.md)).
- **Reserves cruisers** and parks them in formation (installing `AIGoalStaticRoadBlock`/`AIActionStaticRoadBlock`,
  [Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md) on each).
- **Manages the lifecycle** — approach, engage, averted/dodged.

That it's 62 methods for a 168-byte object tells you most of its logic is *siting and formation* (computing where
and how to place the block), with little persistent state — a lot of computation, little memory. Roadblocks are a
[Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md) topic in full.

## AITrafficManager: the ambient population

**`AITrafficManager`** (29 methods, 948 B — the *largest manager by state*) owns the **ambient traffic**: the
civilian-car population, density targets per zone, and the spawn/despawn ring around the camera. Every
`AIVehicleTraffic` on `AIGoalTraffic` ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)) is one of its
charges. Its large state (948 B) holds the population and density bookkeeping for a whole city's worth of traffic,
but few methods (29) because managing traffic is mostly *spawn here, despawn there, keep the density* — a lot of
data, simple logic. It's the counterpart to `AICopManager` for the *civilian* side of the world.

## Why managers as separate singletons

Separating the coordinators from the brains ([C47.1](01-aivehicle-hierarchy.md)) is a clean architecture:

- **Brains drive; managers coordinate.** An `AIVehicleCopCar` drives *its* car; `AICopManager` decides *which* cop
  cars exist and `AIPursuit` decides *what they're chasing*. Each brain stays focused on driving; the cross-car
  logic lives in the managers.
- **One authority per concern.** One `AITrafficManager` for all traffic, one `AICopManager` for all cops, one
  `AIPursuit` per chase — a single owner of each population/activity, avoiding conflicting decisions.
- **Managed on a list.** The managers register on `0x0092C668` and tick each frame
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) — the world's coordination heartbeat.

So the AI has two tiers: **brains** (per-car driving) and **managers** (cross-car coordination). This is why a
city-wide pursuit is coherent — the managers hold the global picture (the fleet, the chase state, the traffic) that
no single brain could, and direct the brains accordingly.

## RE implications

- **Managers coordinate across cars** — `AIPursuit` (98, one chase), `AICopManager` (43, all cops),
  `AITrafficManager` (29, traffic), `AIRoadBlock` (62) — on list `0x0092C668`.
- **`AIPursuit` is the chase director** — roster, Heat, timers, escalation ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **`AICopManager` is the fleet** (supply) vs. `AIPursuit` (tactics) — together "how cops work"
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).
- **Two tiers** — brains (drive) and managers (coordinate) — is what makes a city-wide pursuit coherent.

---

### Key takeaways

- Above the driver brains sit **managers** — coordinators on list `0x0092C668`, all verified vtables: `AIPursuit`
  (98), `AICopManager` (43), `AIRoadBlock` (62), `AITrafficManager` (29).
- **`AIPursuit`** is the **chase director** — a small state machine + rosters owning one pursuit's cops, Heat,
  timers, and escalation ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **`AICopManager`** is the **fleet** (supply — how many cops, spawned where) vs. `AIPursuit` (tactics — what they
  do); together, "how cops work."
- **`AITrafficManager`** (largest manager state, 948 B) owns the ambient population; **`AIRoadBlock`** sites and
  forms roadblocks.
- The AI has **two tiers** — brains (per-car driving) and managers (cross-car coordination) — which is what makes a
  city-wide pursuit coherent.

**Continue:** [C47.4 — Navigation & the named systems](04-navigation-systems.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md)
