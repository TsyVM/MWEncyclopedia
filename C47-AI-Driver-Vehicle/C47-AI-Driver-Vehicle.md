# Chapter 47 — AI Driver Brain & Vehicle Hierarchy

> **Goal of this chapter:** decode the driver brain — the `AIVehicle*` class hierarchy (base `AIVehicle`, 351
> methods — the most method-rich class in the game; plus CopCar, Racecar, Traffic, Helicopter, Human, Empty), the
> profound fact that the *player's* car is an `AIVehicleHuman`, and the manager/coordinator layer (`AIPursuit`,
> `AICopManager`, `AITrafficManager`) and navigation services (`PathFinder`, `Gps`) that direct them.

The goal/action system ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) is *what an AI decides*;
this chapter is *who holds the wheel and who gives the orders*. The **`AIVehicle*`** classes are the driver brains
— the `BEHAVIOR_MECHANIC_AI` implementations ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) that own a car's
goal/action planner and convert its chosen action into throttle/brake/steer. Above them, the **managers**
coordinate who exists, what they intend, and how the world is wired. Together they are the complete AI driver
stack — verified, class by class, against `speed.exe`.

> **Verified against the executable.** The seven `AIVehicle*` brains are byte-verified vtables: `AIVehicle`
> (`0x15CB4F07`, vtable `0x00891998`, **351 methods** — the most in the game, 1852 B), `AIVehicleEmpty` (same
> vtable, driverless), `AIVehicleCopCar` (`0x9F128A92`, `0x008923E8`, **324**, 1964 B), `AIVehicleRacecar`
> (`0x628F3716`, `0x008925B4`, 209, 1996 B), `AIVehicleTraffic` (`0xEDE26480`, `0x00891C08`, 195, 1860 B),
> `AIVehicleHelicopter` (`0x4F76621E`, `0x00891F18`, 196, 2240 B), `AIVehicleHuman` (`0x73A9594F`, `0x0089293C`,
> 133, 2028 B). The managers are verified too: `AIPursuit` (`0x00892770`, **98**), `AICopManager` (`0x008918D8`,
> 43), `AIRoadBlock` (`0x00892130`, 62), `AITrafficManager` (`0x00890F18`, 29), `AvoidableManager` (`0x00890CDC`,
> 17), `Gps` (`0x008911CC`, 16), `PathFinder` (`0x008B5C50`, 16, with `AStarNodeSlotPool`/`AStarSearchSlotPool`).
> Brains register on the mechanics list `0x0092C660`; managers on `0x0092C668`; named systems (`World`,
> `VehicleSystem`, `WRoadNetwork`, `CameraAI`, `SceneryModel`) on `0x00988DFC`.

---

## Deep-dive pages

- [C47.1 — The AIVehicle hierarchy](01-aivehicle-hierarchy.md): the seven driver brains and their fidelities.
- [C47.2 — The player is an AI](02-player-is-ai.md): `AIVehicleHuman`, the brain state, and the unification.
- [C47.3 — The managers](03-managers.md): `AIPursuit`, `AICopManager`, `AITrafficManager`, `AIRoadBlock`.
- [C47.4 — Navigation & the named systems](04-navigation-systems.md): `PathFinder`/`Gps`/`WRoadNetwork`, the world
  roots.
- [C47.5 — Reading the driver brain in RE](05-reading-ai-brain.md): navigating the AI stack.

---

## 47.1 The AIVehicle hierarchy

The driver brain is the **`AIVehicle*`** family ([C47.1](01-aivehicle-hierarchy.md)) — the largest classes in the
game. `AIVehicle` is the base, with **351 methods** (the most method-rich vtable in `speed.exe`): everything a
controlled vehicle can do — sense, plan, steer, react — is declared here. The specialisations override slices:
`AIVehicleCopCar` (324, the pursuit toolkit), `AIVehicleRacecar` (209, racing), `AIVehicleTraffic` (195, lane-keep),
`AIVehicleHelicopter` (196, flight), `AIVehicleHuman` (133, the player). All register on the mechanics list
`0x0092C660`.

## 47.2 The player is an AI

The profound fact ([C47.2](02-player-is-ai.md)): **the player's car is an `AIVehicleHuman`** — it runs the *same
brain wiring* as every AI car, but takes its commands from `InputPlayer`
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) instead of a goal/action planner. This is why the registry has
*no separate "player vehicle" class*. The player and the AI are the same kind of driver, differing only in the
command source — the class-level twin of the input/AI mechanic swap
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)).

## 47.3 The managers

Above the brains are the **managers** ([C47.3](03-managers.md)) — coordinators that decide who exists and what they
intend, on list-head `0x0092C668`. **`AIPursuit`** (98 methods) is the pursuit director — it owns a chase's state
(assigned cops, Heat, timers) and escalates each cop's goal ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
**`AICopManager`** (43) is the fleet manager — the roster of all cops, spawning against the Heat tables
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)). **`AITrafficManager`** (29, largest
manager by state) owns the ambient population. **`AIRoadBlock`** (62) orchestrates roadblocks.

## 47.4 Navigation & the named systems

The AI navigates via **`PathFinder`** ([C47.4](04-navigation-systems.md)) — an **A\* search** over the road graph,
verified by its `AStarNodeSlotPool`/`AStarSearchSlotPool` (pre-sized pools, no per-query allocation). **`Gps`**
routes over the same graph for the player's direction arrow; **`AvoidableManager`** answers "what to steer around."
All run over **`WRoadNetwork`** — the runtime road graph (from CARP,
[Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)), one of five **named singleton systems** (`World`,
`VehicleSystem`, `WRoadNetwork`, `CameraAI`, `SceneryModel`) on list `0x00988DFC` — the roots of the object graph.

---

### Key takeaways

- The driver brain is the **`AIVehicle*`** family — base `AIVehicle` (**351 methods**, the most in the game),
  plus `CopCar` (324), `Racecar` (209), `Traffic` (195), `Helicopter` (196), `Human` (133), `Empty` — verified
  vtables/sizes.
- **The player's car is an `AIVehicleHuman`** — the same brain, commanded by `InputPlayer` not a planner; there's
  no separate player-vehicle class.
- **Managers coordinate** the brains — `AIPursuit` (98, the chase director), `AICopManager` (43, the fleet),
  `AITrafficManager` (29, traffic), `AIRoadBlock` (62) — on list `0x0092C668`.
- The AI **navigates** via `PathFinder` (A\* with pre-sized pools), `Gps`, and `AvoidableManager` over the
  **`WRoadNetwork`** road graph.
- Five **named singleton systems** (`World`, `VehicleSystem`, `WRoadNetwork`, `CameraAI`, `SceneryModel`) on
  `0x00988DFC` are the roots of the object graph.

**Next:** [Chapter 48 — Pursuit & Heat: the State Machine](../C48-Pursuit-Heat/C48-Pursuit-Heat.md): how `AIPursuit`
runs a chase.
