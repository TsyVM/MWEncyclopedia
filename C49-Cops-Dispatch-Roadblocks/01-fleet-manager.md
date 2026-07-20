# C49.1 — AICopManager & the OnTask Pipeline

> **The one-sentence version:** `AICopManager` (43 methods) runs a per-frame `OnTask` pipeline — UpdatePatrols
> (`0x4314C0`) → UpdatePursuits (`0x43E8D0`) → UpdateRoadBlocks (`0x439EE0`) → UpdateSpawnRequests (`0x42E8D0`),
> plus ApplyBreakerZones (`0x42E930`) — the fleet's heartbeat that keeps the right cops on you.

[← Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md) · [Next: C49.2 — The cop roster →](02-cop-roster.md)

---

## The fleet's heartbeat

`AICopManager` ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)) owns *all the cops*, and its work is a
**per-frame `OnTask` pipeline** — a fixed sequence of sub-updates run each tick
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)). Every sub-update is a real, byte-verified
function:

```
AICopManager::OnTask (each frame):
   ├─ (debug drain)        0x406500  — a drain-and-discard queue stub
   ├─ ApplyBreakerZones    0x42E930  — timed wreck-spheres (pursuit breakers, C49.5)
   ├─ UpdatePatrols        0x4314C0  — promote idle patrols that spot you
   ├─ UpdatePursuits       0x43E8D0  — drive the active chase roster
   ├─ UpdateRoadBlocks     0x439EE0  — orchestrate roadblocks (C49.4)
   └─ UpdateSpawnRequests  0x42E8D0  — fulfil pending spawns, retry failures
```

This pipeline is the fleet's heartbeat: each frame, the manager checks which patrols should escalate, advances the
chase, manages roadblocks, and spawns any cops the dispatch tables ([C49.3](03-formations-dispatch.md)) call for. It
runs whether or not you're in a pursuit — patrols are always being watched for triggers.

> ✅ *Verified:* the `OnTask` sub-updates are byte-verified functions in `speed.exe` — UpdatePatrols `0x4314C0`
> (`6A FF …` SEH prologue), UpdatePursuits `0x43E8D0` (`6A FF 68 28 95 86 …`), UpdateRoadBlocks `0x439EE0`
> (`6A FF 68 88 8D 86 …`), UpdateSpawnRequests `0x42E8D0` (`56 8B F1 8B 46 6C` — reads `[esi+0x6C]`),
> ApplyBreakerZones `0x42E930` (`83 EC 10 57 8B F9`).

## UpdatePatrols: the trigger

**UpdatePatrols** (`0x4314C0`) is where a peaceful drive becomes a pursuit. It polls each patrol cruiser
([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md), `AIGoalPatrol`) — checking whether it has *spotted* you
committing an infraction (speeding, ramming, being wanted). When a patrol spots you:

- It's promoted from `AIGoalPatrol` to `AIGoalPursuit` ([Chapter 48](../C48-Pursuit-Heat/03-escalation-ladder.md)) —
  the chase begins.
- The spotted-target pointer is written for `UpdatePursuits` ([below](#updatepursuits-the-chase)) to consume.

So UpdatePatrols is the *sensor* of the fleet — the sweep that turns an ambient cruiser into an active pursuer the
instant you cross it wrong. A patrol drives as traffic until this update flips it into the chase. This is why cops
seem to "notice" you: UpdatePatrols is polling them against your infractions every frame.

> 🟡 *Reasoned:* the patrol-spots-perp promotion is the natural reading of UpdatePatrols writing a spotted-target
> pointer consumed by UpdatePursuits and the `AIGoalPatrol`→`AIGoalPursuit` swap
> ([Chapter 48](../C48-Pursuit-Heat/03-escalation-ladder.md)); the exact spot conditions are pursuit vault data.
> The functions and their ordering are verified.

## UpdatePursuits: the chase

**UpdatePursuits** (`0x43E8D0`) drives the *active* pursuit — it consumes the spotted-target pointer that
UpdatePatrols wrote and advances the chase roster: which cops are assigned, whether the count matches the Heat
target ([C49.3](03-formations-dispatch.md)), and whether to gate spawning against the fleet cap. It's the bridge
between the fleet manager (`AICopManager`) and the chase director (`AIPursuit`,
[Chapter 48](../C48-Pursuit-Heat/01-the-cast.md)) — the manager's per-frame servicing of the pursuit's needs. If
the pursuit wants more cops than are present, UpdatePursuits (with UpdateSpawnRequests) makes up the difference,
capped so the fleet never exceeds the Heat's limit.

## UpdateSpawnRequests: the supply

**UpdateSpawnRequests** (`0x42E8D0`) is the *supply valve* — verified to read a **pending spawn queue at
`[esi+0x6C]`** and fulfil it, retrying failed spawns next tick. When the pursuit needs more cops
([C49.3](03-formations-dispatch.md)), a spawn request is queued; this update dequeues and attempts it (finding a
valid off-screen spawn point, constructing the cop). Successful spawns bump the fleet counter that UpdatePursuits
gates against its cap. So spawning is *asynchronous and retried*: a request that can't be fulfilled this frame (no
valid spawn point nearby) is retried until it can — which is why reinforcements arrive with a slight, natural delay
rather than popping in instantly.

## Why a pipeline

Structuring the fleet as a fixed `OnTask` pipeline ([above](#the-fleets-heartbeat)) is clean, verifiable engine
design:

- **Deterministic order.** Patrols are checked before pursuits (a newly-spotted cop joins this frame's chase);
  roadblocks and spawns are serviced after. The order encodes the dependencies.
- **Each phase is a function.** UpdatePatrols, UpdatePursuits, etc. are separate, testable functions — the pipeline
  is legible in the disassembly ([C49.6](06-reading-fleet.md)) as a sequence of calls.
- **One heartbeat.** The whole fleet advances once per frame in a known order — no scattered, order-dependent
  updates. The manhunt is coherent because it ticks as one pipeline.

So `AICopManager`'s pipeline is the fleet made into a per-frame routine: sense (patrols), chase (pursuits), block
(roadblocks), supply (spawns), all in order. Reading these five functions *is* reading how the cops operate as a
force.

## RE implications

- **`AICopManager::OnTask` is a verified pipeline** — UpdatePatrols → UpdatePursuits → UpdateRoadBlocks →
  UpdateSpawnRequests (+ ApplyBreakerZones).
- **UpdatePatrols** (`0x4314C0`) is the *sensor* — promotes patrols that spot you to pursuit.
- **UpdateSpawnRequests** (`0x42E8D0`) is the *supply valve* — reads the spawn queue `[esi+0x6C]`, retries
  failures.
- **A fixed pipeline** gives deterministic, legible, coherent fleet behaviour — one heartbeat per frame.

---

### Key takeaways

- `AICopManager` runs a verified per-frame **`OnTask` pipeline**: **UpdatePatrols → UpdatePursuits →
  UpdateRoadBlocks → UpdateSpawnRequests**, plus **ApplyBreakerZones**.
- **UpdatePatrols** (`0x4314C0`) is the fleet's **sensor** — it polls patrols and promotes any that spot you into
  the pursuit (the moment a chase begins).
- **UpdatePursuits** (`0x43E8D0`) services the active chase, gating cop count against the Heat cap.
- **UpdateSpawnRequests** (`0x42E8D0`) is the **supply valve** — reads the spawn queue (`[esi+0x6C]`), fulfils and
  **retries** spawns, so reinforcements arrive with a natural delay.
- The fixed pipeline gives **deterministic, legible, coherent** fleet behaviour — reading these five functions is
  reading how cops operate as a force.

**Continue:** [C49.2 — The cop roster](02-cop-roster.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md)
