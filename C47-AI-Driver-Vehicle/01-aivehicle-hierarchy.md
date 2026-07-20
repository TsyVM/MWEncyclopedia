# C47.1 — The AIVehicle Hierarchy

> **The one-sentence version:** the driver brain is the `AIVehicle*` family — base `AIVehicle` (vtable
> `0x00891998`, **351 methods**, the most in the game), specialised into CopCar (324), Racecar (209), Traffic
> (195), Helicopter (196), Human (133), and Empty — the largest classes in the whole engine.

[← Chapter 47 hub](C47-AI-Driver-Vehicle.md) · [Next: C47.2 — The player is an AI →](02-player-is-ai.md)

---

## The base: AIVehicle (351 methods)

The driver brain is the **`AIVehicle`** class — and it is the **most method-rich vtable in the entire game**:
**351 methods** at `0x00891998`, over 1852 bytes of state. This makes sense — driving a car well is the engine's
hardest job, and `AIVehicle` declares *everything* a controlled vehicle can do:

- **Sense the world** — perceive the road, the target, obstacles, other cars.
- **Plan** — hold and run the goal/action system ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)).
- **Steer** — convert the chosen action into throttle/brake/steer for the engine and rigid body
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
- **React** — handle being stuck, airborne, damaged, off-route.

The specialisations ([below](#the-seven-brains)) *override slices* of these 351 methods — each keeps most of the
base and replaces the parts specific to its role. So `AIVehicle` is the substrate of all driving intelligence, and
its 351 methods are the vocabulary the whole AI is built from.

> ✅ *Verified:* `AIVehicle` is a real vtable at `0x00891998` with exactly **351 methods** (confirmed by pointer
> count — the largest in `speed.exe`); hash `0x15CB4F07`, 1852 B. It registers on the mechanics list `0x0092C660`
> (the `BEHAVIOR_MECHANIC_AI` slot, [Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).

## The seven brains

The `AIVehicle*` family is seven classes, all verified vtables, one per driver role:

| Class | Hash | vtable | Methods | Size | Role |
|---|---|---|---|---|---|
| `AIVehicle` | `0x15CB4F07` | `0x00891998` | **351** | 1852 B | base driver brain |
| `AIVehicleEmpty` | `0x8ED4136F` | `0x00891998` | 351 | 1852 B | driverless (idle planner) |
| `AIVehicleCopCar` | `0x9F128A92` | `0x008923E8` | 324 | 1964 B | the cop brain |
| `AIVehicleRacecar` | `0x628F3716` | `0x008925B4` | 209 | 1996 B | the race-opponent brain |
| `AIVehicleTraffic` | `0xEDE26480` | `0x00891C08` | 195 | 1860 B | the civilian brain |
| `AIVehicleHelicopter` | `0x4F76621E` | `0x00891F18` | 196 | 2240 B | the police-helicopter brain |
| `AIVehicleHuman` | `0x73A9594F` | `0x0089293C` | 133 | 2028 B | the player-car brain ([C47.2](02-player-is-ai.md)) |

These are **entities by size** — the largest classes in the registry (1852–2240 bytes each), because a driver
carries a lot of state: perception, current goal/action, targets, timers, route context
([C47.2](02-player-is-ai.md)).

## AIVehicleEmpty: the shared-vtable proof

A revealing case: **`AIVehicleEmpty` shares `AIVehicle`'s exact vtable and size** (`0x00891998`, 351 methods, 1852
B). This is the verified proof that `AIVehicleEmpty` is *`AIVehicle` with the planner idle* — a vehicle that exists
physically but has no active controller (parked, disabled, or awaiting assignment), not a new behaviour. The shared
vtable ([C46.3](../C46-AI-Goals-Actions/03-data-only-goals.md) used the same technique for goals) means identical
code; "Empty" is a *state* of the base brain, not a different class of brain. This is the kind of fact only
byte-level verification reveals — the name suggests a distinct thing, but the shared vtable proves it's the base
idling.

## CopCar and Racecar: the two big overrides

The two most heavily-overriding specialisations are the cop and the racer — the two most demanding driving roles:

- **`AIVehicleCopCar` (324 methods, 1964 B — the... near-largest).** The **cop brain** overrides the base with the
  full pursuit toolkit: awareness of the suspect, the pursuit goal set (Pit/Ram/PullOver/HeadOnRam,
  [Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), formation behaviour under `AICopManager`
  ([C47.3](03-managers.md)), and roadblock participation. To understand "how cops think," this class plus
  `AIPursuit` ([C47.3](03-managers.md)) is the pair.
- **`AIVehicleRacecar` (209 methods, 1996 B).** The **race-opponent brain** drives `AIGoalRacer`
  ([C46.4](../C46-AI-Goals-Actions/04-override-goals.md)): line choice, drafting, catch-up/rubber-band, rival
  awareness. Fewer overrides than the cop — racing is demanding but *narrower* than pursuit (a racer just races; a
  cop chases, rams, boxes, blocks, and formates).

So the method counts rank the roles by breadth: cop (324, the broadest — many tactics) > racer (209, focused) >
helicopter (196) / traffic (195) > human (133, [C47.2](02-player-is-ai.md)). The cop is the most complex driver in
the game because pursuit is the most multi-faceted behaviour.

## RE implications

- **`AIVehicle` (351 methods)** is the base driver brain — the most method-rich class in the game; the others
  override slices.
- **Seven brains** — base, Empty, CopCar (324), Racecar (209), Traffic (195), Helicopter (196), Human (133) — the
  largest classes (1852–2240 B).
- **`AIVehicleEmpty` shares the base vtable** — verified proof it's `AIVehicle` idling, not a new behaviour.
- **CopCar (324) is the broadest override** — pursuit is the most multi-faceted driving role.

---

### Key takeaways

- The driver brain is the **`AIVehicle*`** family; base **`AIVehicle`** has **351 methods** (verified) — the
  **most method-rich vtable in the game** — declaring everything a controlled car can do.
- Seven brains, all verified vtables, are the **largest classes** in the engine (1852–2240 B): base, Empty, CopCar
  (324), Racecar (209), Traffic (195), Helicopter (196), Human (133).
- **`AIVehicleEmpty` shares `AIVehicle`'s exact vtable** — byte-level proof it's the base brain *idling*
  (driverless), not a new class.
- **`AIVehicleCopCar` (324)** is the broadest override — the cop is the most complex driver because pursuit has the
  most facets (chase, ram, box, block, formate).
- The method counts **rank the roles by breadth** — cop > racer > heli ≈ traffic > human.

**Continue:** [C47.2 — The player is an AI](02-player-is-ai.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md)
