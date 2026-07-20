# Chapter 49 — Cops: Dispatch, Formations, Roadblocks & Bust

> **Goal of this chapter:** decode the cop *fleet* — `AICopManager` and its verified per-frame `OnTask` pipeline
> (UpdatePatrols → UpdatePursuits → UpdateRoadBlocks → UpdateSpawnRequests), the cop roster (`cop1`…`copgto`,
> `copheli`), the Heat-indexed dispatch tables (`CopCountRecord`/`CopFormationRecord`, ×22 each), roadblocks
> (`AIRoadBlock` + the `Roadblock_*` lifecycle), and spike strips.

Where [Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md) decoded the pursuit *state machine* (`AIPursuit`, the
tactics of one chase), this chapter decodes the **fleet** — the *supply* side: how `AICopManager` keeps the right
number and mix of cops on you, spawns and forms them up, orchestrates roadblocks, and deploys spikes. It is the
logistics of the manhunt, and — like the rest of the pursuit — it runs a small set of verified code classes over
heavily-referenced vault tables.

> **Verified against the executable and vault.** `AICopManager` (the fleet, vtable `0x008918D8`, 43 methods) runs a
> per-frame `OnTask` pipeline whose sub-updates are byte-verified: **UpdatePatrols** (`0x4314C0`), **UpdatePursuits**
> (`0x43E8D0`), **UpdateRoadBlocks** (`0x439EE0`), **UpdateSpawnRequests** (`0x42E8D0`, reads the spawn queue at
> `[esi+0x6C]`), plus **ApplyBreakerZones** (`0x42E930`). The **cop roster** is named in `speed.exe`: `cop1`,
> `copmidsize`, `copsport`, `copsuv`, `copsuvpatrol`, `copcross`, `copgto`/`COPGTO`, `copheli`, and ghost variants
> (`copghost`, `copgtoghost`). The **dispatch tables** are vault records — `CopCountRecord` (`0xFCAA46E2`, ×22) and
> `CopFormationRecord` (`0xB5A53D76`, ×22). **`AIRoadBlock`** (vtable `0x00892130`, 62 methods) runs the verified
> `Roadblock_*` lifecycle (`CallForRB`, `RBApproach`, `RBEngage`, `RBAverted`, …). Spikes are `SpikeStrip`/
> `SpikeBelt`.

---

## Deep-dive pages

- [C49.1 — AICopManager & the OnTask pipeline](01-fleet-manager.md): the fleet's per-frame heartbeat.
- [C49.2 — The cop roster](02-cop-roster.md): the cop shells and their Heat tiers.
- [C49.3 — Formations & dispatch](03-formations-dispatch.md): `CopCountRecord`/`CopFormationRecord`, spawn requests.
- [C49.4 — Roadblocks](04-roadblocks.md): `AIRoadBlock` and the `Roadblock_*` lifecycle.
- [C49.5 — Spikes & pursuit breakers](05-spikes-breakers.md): `SpikeStrip` and the breaker zones.
- [C49.6 — Reading the fleet in RE](06-reading-fleet.md): navigating the cop system.

---

## 49.1 The fleet manager

**`AICopManager`** ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)) is the fleet — it owns *all the cops* and
runs a per-frame `OnTask` pipeline ([C49.1](01-fleet-manager.md)) of verified sub-updates: **UpdatePatrols**
(promote idle patrols that spot you), **UpdatePursuits** (drive the active chase roster), **UpdateRoadBlocks**
(orchestrate blocks), and **UpdateSpawnRequests** (fulfil pending spawns, retrying failures). Plus
**ApplyBreakerZones** (timed wreck-spheres from pursuit breakers, [C49.5](05-spikes-breakers.md)). This pipeline is
the fleet's heartbeat — the loop that keeps the right cops on you.

## 49.2 The cop roster

The cops come in a **roster of shells** ([C49.2](02-cop-roster.md)) — verified strings: `cop1` (the basic
cruiser), `copmidsize`, `copsport`, `copsuv`/`copsuvpatrol` (the SUVs/Rhinos), `copcross` (cross-country),
`copgto`/`COPGTO` (the named GTO unit — a specific Blacklist pursuit vehicle), and `copheli` (the helicopter). Heat
([Chapter 48](../C48-Pursuit-Heat/02-heat.md)) selects which shells are dispatched — light cruisers low, SUVs and
the chopper high. The roster *is* the escalation you see on screen.

## 49.3 Formations & dispatch

The fleet's size and composition come from the **dispatch tables** ([C49.3](03-formations-dispatch.md)):
`CopCountRecord` (×22 — how many cops at each Heat) and `CopFormationRecord` (×22 — the mix and formation). Heat
indexes these tables ([Chapter 48](../C48-Pursuit-Heat/02-heat.md)); `UpdateSpawnRequests` fulfils the difference
between wanted and present, spawning against a cap. Reinforcement *strategies* — Leader, Heavy, AirSupport —
authorise heavier dispatch at higher Heat.

## 49.4 Roadblocks

**`AIRoadBlock`** ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)) orchestrates roadblocks through a verified
**`Roadblock_*` lifecycle** ([C49.4](04-roadblocks.md)): `CallForRB` (AIPursuit requests one) → site selection →
`RBApproach` → `RBEngage` (the block is live ahead of you) → `RBAverted`/dodged (you got past). The verified stat
keys `roadblocks_dodged`/`roadblocks_dodged_in_pursuit` track your evasions. Cruisers holding a block run
`AIGoalStaticRoadBlock` ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)).

## 49.5 Spikes & breakers

Two more tools tilt a pursuit ([C49.5](05-spikes-breakers.md)): **spike strips** (`SpikeStrip`/`SpikeBelt`) that
deploy across the road to puncture your tyres (`ETirePunctured`,
[Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)), and **pursuit breakers** — droppable
environment (via `ApplyBreakerZones`, `0x42E930`) that creates timed wreck-spheres disabling cops caught inside.
Spikes are the cops' tool against you; breakers are your tool against them.

---

### Key takeaways

- **`AICopManager`** is the fleet — its verified `OnTask` pipeline runs **UpdatePatrols → UpdatePursuits →
  UpdateRoadBlocks → UpdateSpawnRequests** (+ ApplyBreakerZones) each frame.
- The **cop roster** (`cop1`, `copmidsize`, `copsport`, `copsuv`, `copcross`, `copgto`, `copheli`) is Heat-tiered —
  light cruisers low, SUVs and the chopper high.
- **Dispatch is data** — `CopCountRecord` (×22, how many) and `CopFormationRecord` (×22, the mix) indexed by Heat;
  spawn requests fulfil the difference.
- **Roadblocks** run the verified `Roadblock_*` lifecycle (`CallForRB` → `RBApproach` → `RBEngage` →
  `RBAverted`/dodged) via `AIRoadBlock`.
- **Spikes** (`SpikeStrip`) puncture your tyres; **pursuit breakers** (`ApplyBreakerZones`) disable cops — the
  two-way tools of the manhunt.

**Next:** [Chapter 50 — Verification Methodology](../C50-Verification-Methodology/C50-Verification-Methodology.md):
how every claim in this book was proven.
