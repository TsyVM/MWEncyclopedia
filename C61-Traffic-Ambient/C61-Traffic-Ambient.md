# Chapter 61 — Traffic & Ambient World Life

> **Goal of this chapter:** decode the ambient traffic that fills Rockport — `AITrafficManager` (the population
> owner), `TrafficLevel` (density), the spawn system (`SpawnTraffic`, `AIParkedCarSpawner` for moving and parked
> cars), and the `AIVehicleTraffic` behaviour that makes civilian cars lane-follow, swerve, and crash.

A city needs *life* — cars on the roads, parked at the curb, honking and swerving. This chapter decodes Most
Wanted's ambient traffic: how `AITrafficManager` maintains a believable population around the player, spawns and
despawns it as you drive, and drives each civilian car with the cheap `AIVehicleTraffic` brain
([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md)). It's the system that makes Rockport feel *inhabited*
rather than empty — and the obstacles (and occasional weapons) of a pursuit.

> **Verified against the executable.** Traffic is named in `speed.exe`: **`AITrafficManager`**/`TrafficManager`
> (the population owner, [C47.3](../C47-AI-Driver-Vehicle/03-managers.md)), **`TrafficLevel`** (density),
> `TrafficCar`/`TrafficCars`, `TrafficSpeed`, and the sound/effect names (`TrafficEngine`, `TrafficHorn`,
> `TrafficSkids`, `TrafficWoosh`, `SFXCTL_3DTrafficPos`). The **spawn system** — `SpawnTraffic`,
> `AIParkedCarSpawner`, `Spawner`, `SpawnMode` (alongside `SpawnCop`, `SpawnCharacter`,
> [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)). Traffic cars are
> `AIVehicleTraffic` ([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md), 195 methods) on `AIGoalTraffic`
> ([C46.2](../C46-AI-Goals-Actions/02-goal-catalog.md)).

---

## Deep-dive pages

- [C61.1 — The traffic system](01-traffic-system.md): `AITrafficManager` and the population.
- [C61.2 — Traffic density](02-traffic-density.md): `TrafficLevel` and the spawn/despawn ring.
- [C61.3 — Spawning](03-spawning.md): moving (`SpawnTraffic`) and parked (`AIParkedCarSpawner`) cars.
- [C61.4 — Traffic behaviour](04-traffic-behavior.md): `AIVehicleTraffic` — lane-follow, swerve, crash.
- [C61.5 — Reading traffic in RE](05-reading-traffic.md): navigating the traffic system.
- [C61.6 — The track path network](06-traffic-paths.md): the on-disk AI/traffic routes — `TrackPathManager`
  (`0x80034150`) and the `0x34152` waypoint segments, decoded byte-for-byte.

---

## 61.1 The traffic system

**`AITrafficManager`** ([C47.3](../C47-AI-Driver-Vehicle/03-managers.md), 948 B — the largest manager) owns the
*ambient population* ([C61.1](01-traffic-system.md)): the set of civilian cars on the roads, maintained around the
player. It holds the density targets, the spawn/despawn ring, and the roster of active `AIVehicleTraffic`
([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md)) cars — the civilian counterpart of `AICopManager`
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) for the cop fleet.

## 61.2 Traffic density

**`TrafficLevel`** ([C61.2](02-traffic-density.md)) sets *how much* traffic — the density the manager maintains,
from empty roads to a busy city. Traffic is spawned in a **ring around the camera**
([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)): cars appear ahead (out of view), populate the
roads near you, and despawn behind (out of view). So the world is *always* populated around you at the set density,
without simulating the whole city — a moving window of traffic that follows the player.

## 61.3 Spawning

Traffic comes in two kinds ([C61.3](03-spawning.md)): **moving** (`SpawnTraffic` — cars driving the roads) and
**parked** (`AIParkedCarSpawner` — cars at the curb, in lots). Both are spawned by the population system into the
ring ([C61.2](02-traffic-density.md)). The spawn system is shared with cops (`SpawnCop`,
[C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)) and other actors (`SpawnCharacter`, `SpawnSmackable`)
— a general `Spawner` machinery that populates the world with the right actors at the right places.

## 61.4 Traffic behaviour

Each civilian car is an **`AIVehicleTraffic`** ([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md), 195
methods) on `AIGoalTraffic` ([C46.2](../C46-AI-Goals-Actions/02-goal-catalog.md)) — a cheap brain
([C61.4](04-traffic-behavior.md)) that **lane-follows** the road network
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)), **swerves** to avoid obstacles
([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md), `AvoidableManager`), and runs a **crash-state**
machine when hit ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)). Traffic is a real
`RBVehicle` ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) — you can *hit* it, wreck it, and use
it as a pursuit weapon (a spun traffic car blocks cops).

---

### Key takeaways

- **`AITrafficManager`** (the largest manager, 948 B) owns the **ambient population** — the civilian counterpart of
  the cop fleet manager.
- **`TrafficLevel`** sets the **density**; traffic is maintained in a **ring around the camera** — a moving window
  that keeps the world populated without simulating the whole city.
- Traffic is **moving** (`SpawnTraffic`) or **parked** (`AIParkedCarSpawner`), spawned by a shared **`Spawner`**
  machinery (also `SpawnCop`, `SpawnCharacter`).
- Each civilian car is an **`AIVehicleTraffic`** (195 methods) on `AIGoalTraffic` — lane-following, swerving, and
  crash-reacting, a **real `RBVehicle`** you can hit and wreck.
- Traffic makes Rockport feel **inhabited** — and is an **obstacle and weapon** in pursuits (a wrecked traffic car
  blocks cops).

**Next:** [Chapter 62 — Physics Constraints, Joints & Trailers](../C62-Constraints-Joints/C62-Constraints-Joints.md):
bodies linked together.
