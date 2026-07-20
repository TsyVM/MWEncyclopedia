# C47.5 — Reading the Driver Brain in RE

> **The one-sentence version:** navigate the AI stack by the `AIVehicle*` brain vtables (base `0x00891998`/351),
> the manager vtables on `0x0092C668`, the navigation services, and the named systems on `0x00988DFC` — reading
> the AI as brains (drive) under managers (coordinate) over shared world systems.

[← C47.4 — Navigation & the named systems](04-navigation-systems.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md) ·
[Next: Chapter 48 — Pursuit & Heat: the State Machine →](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)

---

## Anchors for AI-stack RE

The AI stack is anchored on verified vtables and list-heads:

- **The brain vtables** — `AIVehicle` `0x00891998`/351, `AIVehicleCopCar` `0x008923E8`/324, etc.
  ([C47.1](01-aivehicle-hierarchy.md)); brains on mechanics list `0x0092C660`.
- **The manager vtables** — `AIPursuit` `0x00892770`/98, `AICopManager` `0x008918D8`/43, etc.
  ([C47.3](03-managers.md)); managers on `0x0092C668`.
- **The navigation services** — `PathFinder` `0x008B5C50`, `Gps` `0x008911CC`, `AvoidableManager` `0x00890CDC`
  ([C47.4](04-navigation-systems.md)).
- **The named systems** — `World`, `VehicleSystem`, `WRoadNetwork`, `CameraAI`, `SceneryModel` on `0x00988DFC`
  ([C47.4](04-navigation-systems.md)).

From these, the whole AI stack is navigable: brains, managers, navigation, and roots.

## The three-list map

The AI stack maps cleanly onto **three list-heads** ([C33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)):

```
0x0092C660  mechanics    → the AIVehicle* driver brains (+ the vehicle mechanics, Ch 40)
0x0092C668  managers     → AIPursuit, AICopManager, AITrafficManager, AIRoadBlock, ...
0x00988DFC  named systems→ World, VehicleSystem, WRoadNetwork, CameraAI, SceneryModel
```

Walking these three lists ([C32.6](../C32-Runtime-Class-System/06-reading-binary.md)) enumerates the entire AI
stack: the brains that drive, the managers that coordinate, and the systems that root the world. This is the
top-down map of AI in one view — brains under managers over systems. Reading the three lists tells you every AI
class the game has and which tier it belongs to.

## Verifying by method count

As throughout the vehicle chapters, the decisive class-verification is the **method count**
([C42.6](../C42-Suspension-Tyres-Drivetrain/06-reading-drivetrain.md)): a claimed AI class is real if its vtable is
a clean run of code pointers of the stated length. Every class in this chapter was confirmed this way:

```
351  AIVehicle          (base — the most in the game)
324  AIVehicleCopCar
209  AIVehicleRacecar
196  AIVehicleHelicopter
195  AIVehicleTraffic
133  AIVehicleHuman
 98  AIPursuit
 62  AIRoadBlock
 43  AICopManager
 29  AITrafficManager
 16  PathFinder / Gps
```

The counts are also a **complexity map**: the driver brains (351–133) dwarf the managers (98–29) and services (16)
— because *driving* is far more code than *coordinating* or *routing*. The single biggest class in the game is
`AIVehicle` (351): the engine spends its most code on the act of driving a car well. Reading the counts tells you
where the intelligence — and the RE effort — belongs.

## The complete AI picture

Pulling Chapters 46–47 together, the complete AI is a **four-level stack**:

```
named systems (World, VehicleSystem, WRoadNetwork)   ← the world & its roads
      ↑
managers (AIPursuit, AICopManager, AITrafficManager) ← who exists, what they intend
      ↑
driver brains (AIVehicle*, 351 methods)              ← who holds the wheel
      ↑
goals & actions (Ch 46)                              ← what each brain decides, per tick
```

A car is driven by an `AIVehicle*` brain, which runs a goal/action planner
([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), under a manager that sets its goal and decides
whether it exists, over the world systems that provide the roads and the pool. From the player pressing accelerate
([C47.2](02-player-is-ai.md)) to a cop being dispatched to intercept
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)), everything on the road is one
coherent stack. This is the AI in full — verified, class by class, from `speed.exe`.

## AI grounds the pursuit chapters

With the driver brain and managers decoded, the pursuit chapters
([48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
stand on solid ground: **pursuit** ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) is `AIPursuit` running a
chase's state machine over `AIVehicleCopCar` brains; **cop dispatch**
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) is `AICopManager` spawning and
forming the fleet. The brains ([C47.1](01-aivehicle-hierarchy.md)) and managers ([C47.3](03-managers.md)) of this
chapter are the *actors*; the next two chapters are the *drama* they perform — the pursuit.

## RE implications

- **Anchor on** the brain vtables (`0x0092C660`), manager vtables (`0x0092C668`), navigation services, and named
  systems (`0x00988DFC`).
- **Three list-heads** map the AI stack — brains, managers, named systems.
- **Method count verifies a class** and maps complexity — brains (351–133) ≫ managers (98–29) ≫ services (16).
- **The AI is a four-level stack** — systems → managers → brains → goals/actions.

---

### Key takeaways

- The AI stack is anchored on the **brain vtables** (`0x0092C660`), **manager vtables** (`0x0092C668`),
  **navigation services**, and **named systems** (`0x00988DFC`) — all verified.
- **Three list-heads** map the whole stack — walk them to enumerate every AI class and its tier (brain / manager /
  system).
- The **method-count map** verifies each class and shows the complexity: driver brains (351–133) ≫ managers
  (98–29) ≫ services (16) — driving is the most code.
- The complete AI is a **four-level stack** — named systems → managers → driver brains → goals/actions — one
  coherent chain from the world down to the per-tick decision.
- These brains and managers are the **actors**; the pursuit chapters
  ([48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
  are the **drama** they perform.

**Next:** [Chapter 48 — Pursuit & Heat: the State Machine](../C48-Pursuit-Heat/C48-Pursuit-Heat.md): how `AIPursuit`
runs a chase.

**Sources:** `speed.exe` (verified: `AIVehicle*` brain vtables/method counts — `AIVehicle` `0x00891998`/351,
`AIVehicleCopCar` `0x008923E8`/324, `AIVehicleRacecar` `0x008925B4`/209, `AIVehicleTraffic` `0x00891C08`/195,
`AIVehicleHelicopter` `0x00891F18`/196, `AIVehicleHuman` `0x0089293C`/133; manager vtables `AIPursuit`
`0x00892770`/98, `AICopManager` `0x008918D8`/43, `AIRoadBlock` `0x00892130`/62, `AITrafficManager` `0x00890F18`/29,
`AvoidableManager` `0x00890CDC`/17, `Gps` `0x008911CC`/16, `PathFinder` `0x008B5C50`/16; `AStarNodeSlotPool`/
`AStarSearchSlotPool`; named systems `World`/`VehicleSystem`/`WRoadNetwork`/`CameraAI`/`SceneryModel`; `AIVehicle`
base offsets `+0x38`/`+0x84`/`+0x6C4`/`+0x6EC`).
