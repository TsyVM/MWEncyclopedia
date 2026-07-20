# C49.6 — Reading the Fleet in RE

> **The one-sentence version:** navigate the cop fleet by `AICopManager`'s verified `OnTask` sub-functions, the cop
> roster strings, the dispatch tables (`CopCountRecord`/`CopFormationRecord`), and the `Roadblock_*` lifecycle
> events — reading the manhunt as a per-frame pipeline over Heat-indexed data.

[← C49.5 — Spikes & pursuit breakers](05-spikes-breakers.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md) ·
[Next: Chapter 50 — Verification Methodology →](../C50-Verification-Methodology/C50-Verification-Methodology.md)

---

## Anchors for fleet RE

The cop fleet is anchored on verified structures:

- **`AICopManager`** (`0x008918D8`, 43 methods) and its `OnTask` sub-functions — UpdatePatrols `0x4314C0`,
  UpdatePursuits `0x43E8D0`, UpdateRoadBlocks `0x439EE0`, UpdateSpawnRequests `0x42E8D0`, ApplyBreakerZones
  `0x42E930` ([C49.1](01-fleet-manager.md)).
- **The cop roster** — `cop1`/`copmidsize`/`copsuv`/`copcross`/`copgto`/`copheli` ([C49.2](02-cop-roster.md)).
- **The dispatch tables** — `CopCountRecord` (×22), `CopFormationRecord` (×22) ([C49.3](03-formations-dispatch.md)).
- **The roadblock lifecycle** — `Roadblock_CallForRB`/`RBApproach`/`RBEngage`/`RBAverted`
  ([C49.4](04-roadblocks.md)).
- **The tools** — `SpikeStrip`, `ApplyBreakerZones` ([C49.5](05-spikes-breakers.md)).

From these, the whole fleet is navigable: the pipeline, the roster, the dispatch data, and the set-pieces.

## The RE workflow

Reading the fleet:

1. **Trace the pipeline** — `AICopManager::OnTask` and its five sub-functions ([C49.1](01-fleet-manager.md)); their
   order is the fleet's per-frame logic.
2. **Recover the roster** — grep `cop*` shells ([C49.2](02-cop-roster.md)); the strings are every cruiser type.
3. **Read the dispatch tables** — `CopCountRecord`/`CopFormationRecord` ([C49.3](03-formations-dispatch.md)) rows
   per Heat.
4. **Follow the roadblock events** — the `Roadblock_*` lifecycle ([C49.4](04-roadblocks.md)) marks the set-piece
   states.

The output is the full fleet picture: how cops are supplied, composed, and deployed.

## The named functions are the documentation

A striking feature of the cop fleet ([C49.1](01-fleet-manager.md)) is how *self-documenting* the code is — the
sub-functions and events are named exactly what they do:

- **`UpdatePatrols`, `UpdatePursuits`, `UpdateRoadBlocks`, `UpdateSpawnRequests`** — the pipeline reads like a
  table of contents.
- **`Roadblock_CallForRB`, `RBApproach`, `RBEngage`, `RBAverted`** — the roadblock lifecycle is spelled out.
- **`roadblocks_dodged`, `tire_spikes_dodged`** — even the *stats* are named.

This is because the pursuit system carries its **event/debug string namespace** into the shipped executable
([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) — the same strings that drive cop chatter and scripted beats.
So reverse-engineering the fleet is unusually direct: the strings *narrate* the system. Grep `Roadblock_` and you
have the roadblock state machine; grep `Update` near `AICopManager` and you have its pipeline. The verification-first
discipline ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) is easy here because the
engine labelled its own machinery.

## The pursuit stack is complete

With the fleet decoded, the **pursuit stack** is complete across three chapters:

- **The AI substrate** ([Chapters 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)–[47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md))
  — goals/actions and the `AIVehicleCopCar` brain.
- **The chase state machine** ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) — `AIPursuit`, Heat,
  escalation, the bust envelope.
- **The fleet** (this chapter) — `AICopManager`, the roster, dispatch, roadblocks, spikes, breakers.

Together they are Most Wanted's signature system in full: the cop brains that drive
([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)), the chase that escalates
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), and the fleet that supplies and deploys them (this
chapter) — a small set of verified code classes over a large body of Heat-indexed vault data. This is the deepest
gameplay system in the book, and it's legible top to bottom.

## Reading a live pursuit

In a memory dump ([C32.6](../C32-Runtime-Class-System/06-reading-binary.md)), a live pursuit is readable:

- **The `AIPursuit`** ([Chapter 48](../C48-Pursuit-Heat/01-the-cast.md)) — its Heat, roster, and timers are the
  chase's state.
- **The `AICopManager`** — its spawn queue (`[+0x6C]`, [C49.1](01-fleet-manager.md)) and fleet count show the
  supply state.
- **The cop bodies** — walk the `AIVehicleCopCar` instances ([Chapter 47](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md))
  to see each cop's goal and target.
- **The breaker zones** — the wreck-sphere list ([C49.5](05-spikes-breakers.md)) shows active pursuit-breaker
  effects.

So a running manhunt is fully inspectable: the director's state, the fleet's supply, each cop's intent, and the
active tools. This is the payoff of the verification-first approach
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) — the pursuit isn't a black box;
it's a set of named, structured objects you can read.

## RE implications

- **Anchor on** `AICopManager`'s `OnTask` sub-functions, the cop roster, the dispatch tables, and the `Roadblock_*`
  events.
- **The RE workflow** — trace the pipeline → recover the roster → read the tables → follow the roadblock events.
- **The named functions are the documentation** — the engine labelled its own machinery.
- **The pursuit stack is complete** — AI substrate + chase state machine + fleet, all verified.

---

### Key takeaways

- The fleet is anchored on **`AICopManager`'s `OnTask` sub-functions** (UpdatePatrols/Pursuits/RoadBlocks/
  SpawnRequests), the **cop roster**, the **dispatch tables**, and the **`Roadblock_*` lifecycle**.
- The RE workflow: **trace the pipeline → recover the roster → read the dispatch tables → follow the roadblock
  events**.
- The cop code is **self-documenting** — `UpdatePatrols`, `Roadblock_RBEngage`, `roadblocks_dodged` name exactly
  what they do (the event namespace shipped in the exe).
- A **live pursuit is inspectable** — the `AIPursuit` state, the `AICopManager` spawn queue, each cop's goal, and
  the breaker zones.
- With the fleet decoded, the **pursuit stack is complete** (Chapters 46–49) — MW's signature system, legible top
  to bottom.

**Next:** [Chapter 50 — Verification Methodology](../C50-Verification-Methodology/C50-Verification-Methodology.md):
how every claim in this book was proven.

**Sources:** `speed.exe` (verified: `AICopManager` `0x008918D8`/43 and `OnTask` sub-fns UpdatePatrols `0x4314C0`,
UpdatePursuits `0x43E8D0`, UpdateRoadBlocks `0x439EE0`, UpdateSpawnRequests `0x42E8D0`, ApplyBreakerZones `0x42E930`;
`AIRoadBlock` `0x00892130`/62; cop roster `cop1`/`copmidsize`/`copsport`/`copsuv`/`copsuvpatrol`/`copcross`/`copgto`/
`copheli`; roadblock lifecycle `Roadblock_CallForRB`/`RBApproach`/`RBEngage`/`RBAverted`; `SpikeStrip`/`SpikeBelt`;
`roadblocks_dodged`); `GLOBAL/attributes.bin` (verified: `CopCountRecord` `0xFCAA46E2` ×22, `CopFormationRecord`
`0xB5A53D76` ×22, `copsuv` ×16, `copheli` ×15, `copcross` ×7).
