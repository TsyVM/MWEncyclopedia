# C48.1 — The Cast & the AIPursuit Director

> **The one-sentence version:** a pursuit is run by `AIPursuit` (98 methods) — the director owning one chase's
> Heat, timers, escalation, and each cop's goal — conducting a cast of `AICopManager` (the fleet),
> `AIRoadBlock` (roadblocks), `AIVehicleCopCar` brains, the helicopter, spike strips, and the audio bridge.

[← Chapter 48 hub](C48-Pursuit-Heat.md) · [Next: C48.2 — Heat →](02-heat.md)

---

## AIPursuit: the director of one chase

At the centre of every pursuit is one **`AIPursuit`** ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)) — the
**director** that owns the *state* of that chase (verified vtable `0x00892770`, 98 methods, 652 B — one of the
heaviest non-entity classes). It's effectively a **small state machine plus rosters**, holding:

- **The roster** — which cops are assigned to this pursuit.
- **The Heat level** ([C48.2](02-heat.md)) — the chase's current intensity.
- **The bust/evade timers** ([C48.4](04-bust-evade.md)) — how close you are to caught or clear.
- **The escalation schedule** ([C48.3](03-escalation-ladder.md)) — when to promote cops and call reinforcements.
- **Each cop's goal** — which `AIGoal*` ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)) each assigned
  cop currently holds.

So `AIPursuit` is *the chase*, as a piece of state: everything that makes this pursuit *this* pursuit — its cops,
its heat, its timers, its escalation — lives in the one `AIPursuit` object. Understanding a pursuit is
understanding `AIPursuit`.

> ✅ *Verified:* `AIPursuit` is a real vtable at `0x00892770` with **98 methods** (652 B); it owns a pursuit's
> Heat, roster, timers, and escalation ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)).

## The supporting cast

`AIPursuit` conducts a cast, each a verified class with a role:

| Class | Role in a pursuit |
|---|---|
| `AIPursuit` | **director** of one chase — Heat, timers, escalation, cop goals |
| `AICopManager` | the **fleet** — spawns/despawns units vs. the Heat tables ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) |
| `AIRoadBlock` | sites and manages **roadblocks** ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) |
| `AIVehicleCopCar` | the **cop brain** (324 methods, [C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md)) |
| `AIVehicleHelicopter` | **air support** ([C48.3](03-escalation-ladder.md)) |
| spike strips | the deployed **tyre trap** ([C48.4](04-bust-evade.md)) |
| the AI→audio bridge | fleet state → **siren, chatter, music** ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) |
| reinforcement strategies | Leader → Heavy → AirSupport ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) |

`AIPursuit` decides *what happens* (escalate, call a roadblock, promote to ramming); the cast *does it* (the fleet
spawns, the roadblock sites, the cop brains drive, the audio sings). This division — director vs. cast — is the
pursuit's architecture ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)).

## Director vs. fleet: AIPursuit vs. AICopManager

The two coordinators are easy to conflate but have distinct jobs ([C47.3](../C47-AI-Driver-Vehicle/03-managers.md)):

- **`AIPursuit` — tactics (one chase).** Owns the *state* of a single pursuit: its Heat, its assigned cops' goals,
  its bust/evade timers, its escalation. It answers "what should this chase do next?"
- **`AICopManager` — supply (all cops).** Owns *all the cops* in the world: the roster, spawning against the Heat
  tables ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)), dispatching
  reinforcements. It answers "which cops exist and where?"

So `AIPursuit` is the *conductor* of a chase and `AICopManager` is the *supplier* of cops. When Heat rises
([C48.2](02-heat.md)), `AIPursuit` decides "I need more/heavier cops" and `AICopManager` provides them from the
tables. The two work as a pair — tactics asking, supply providing — which is why "how cops work" is *both* classes
together ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).

## Why a dedicated director per chase

Giving each pursuit its own `AIPursuit` director (rather than folding it into the fleet manager) is a clean design:

- **A chase is a stateful thing.** A pursuit has a lifecycle ([C48.4](04-bust-evade.md)) — begins, escalates,
  resolves — with its own Heat, timers, and roster. Encapsulating that in one object makes the chase a coherent,
  inspectable unit.
- **Tactics separate from supply.** The *decisions* of a chase (escalate, promote, call support) are distinct from
  the *logistics* of the fleet (spawn, despawn). Splitting them ([C47.3](../C47-AI-Driver-Vehicle/03-managers.md))
  keeps each focused.
- **It's a small state machine.** `AIPursuit`'s 98 methods are largely the state-machine transitions
  ([C48.3](03-escalation-ladder.md)) — begin, escalate, bust, evade — plus roster management. A dedicated class is
  the natural home for that machine.

So `AIPursuit` is the pursuit *made into an object* — the chase's state and logic in one place, conducting the cast
that carries it out. The rest of this chapter is really an anatomy of `AIPursuit`: its Heat ([C48.2](02-heat.md)),
its escalation ([C48.3](03-escalation-ladder.md)), and its resolution ([C48.4](04-bust-evade.md)).

## RE implications

- **`AIPursuit` (98 methods) is the director of one chase** — a state machine + rosters owning Heat, timers,
  escalation, and cop goals.
- **The cast** — fleet (`AICopManager`), roadblocks (`AIRoadBlock`), cop brains (`AIVehicleCopCar`), heli, spikes,
  audio — is what `AIPursuit` conducts.
- **Director vs. fleet** — `AIPursuit` (tactics, one chase) vs. `AICopManager` (supply, all cops) — a pair.
- **A dedicated director per chase** encapsulates the pursuit's stateful lifecycle.

---

### Key takeaways

- Every pursuit is run by one **`AIPursuit`** (verified vtable `0x00892770`, **98 methods**) — the **director**
  owning that chase's Heat, roster, timers, escalation, and each cop's goal.
- It conducts a **cast**: the fleet (`AICopManager`), roadblocks (`AIRoadBlock`), cop brains (`AIVehicleCopCar`),
  the helicopter, spike strips, and the audio bridge.
- **Director vs. fleet**: `AIPursuit` handles **tactics** (one chase's decisions); `AICopManager` handles
  **supply** (all cops' spawning) — a pair, asking and providing.
- A **dedicated director per chase** encapsulates the pursuit's stateful lifecycle (begin → escalate → resolve) as
  one inspectable object.
- The chapter is essentially an **anatomy of `AIPursuit`** — its Heat, escalation, and resolution.

**Continue:** [C48.2 — Heat: the escalation variable](02-heat.md) · [Chapter 48 hub](C48-Pursuit-Heat.md)
