# C61.5 — Reading Traffic in RE

> **The one-sentence version:** navigate traffic by `AITrafficManager`, `TrafficLevel`, the spawners
> (`SpawnTraffic`/`AIParkedCarSpawner`), and `AIVehicleTraffic` — reading it as a population manager maintaining
> real physics cars in a moving window.

[← C61.4 — Traffic behaviour](04-traffic-behavior.md) · [Chapter 61 hub](C61-Traffic-Ambient.md) ·
[Next: C61.6 — The track path network →](06-traffic-paths.md)

---

## Anchors for traffic RE

The traffic system is anchored on verified strings:

- **`AITrafficManager`** / `TrafficManager` — the population owner ([C61.1](01-traffic-system.md)).
- **`TrafficLevel`** / `TrafficSpeed` — density and speed ([C61.2](02-traffic-density.md)).
- **The spawners** — `SpawnTraffic`, `AIParkedCarSpawner`, the shared `Spawner` ([C61.3](03-spawning.md)).
- **`AIVehicleTraffic`** — the civilian brain ([C61.4](04-traffic-behavior.md)).

From these, traffic is navigable: the manager, the density, the spawners, and the brain.

## The RE workflow

Reading traffic:

1. **Find the manager** — `AITrafficManager` ([C61.1](01-traffic-system.md)) and its density state.
2. **Read the density** — `TrafficLevel` ([C61.2](02-traffic-density.md)) and the spawn ring.
3. **Trace the spawners** — `SpawnTraffic`/`AIParkedCarSpawner` ([C61.3](03-spawning.md)).
4. **Read the brain** — `AIVehicleTraffic`/`AIGoalTraffic` ([C61.4](04-traffic-behavior.md)).

The output is the full traffic picture: manager, density, spawning, and behaviour.

## Traffic completes the world's population

With traffic decoded, the *world's population* is complete across the actor-management chapters:

- **Cops** ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) — `AICopManager`
  populates the pursuit.
- **Traffic** (this chapter) — `AITrafficManager` populates the ambient city.
- **Scenery/smackables** ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md),
  [C43.5](../C43-Collision-Contacts/05-smackables.md)) — the static and knock-over dressing.

Together they fill Rockport with *actors* — the cops chasing you, the traffic you weave through, the props you
smash. All use the same underlying patterns: a population manager
([C61.1](01-traffic-system.md)), a moving window ([C61.2](02-traffic-density.md)), a shared spawner
([C61.3](03-spawning.md)), and pooled actors ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)). So
the world's *life* — everything moving and hittable in it — is these population systems, maintaining the right
actors around the player. Reading traffic completes the picture of *how Rockport is populated*: not a static scene,
but a continuously-managed cast of cops, cars, and props, spawned and despawned in your wake.

## Traffic embodies the open-world economy

The single insight to carry from traffic is the **open-world economy** ([C61.2](02-traffic-density.md)): a *vast*
world is affordable because you only ever pay for the *window* around the player. Traffic is the clearest example —
a whole city of cars, but only your neighbourhood simulated. This principle recurs everywhere:

- **World geometry** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — stream the sections near you.
- **Scenery** ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) — cull to what's visible; smackables sleep
  until hit ([C43.5](../C43-Collision-Contacts/05-smackables.md)).
- **Traffic** (this chapter) — populate the moving window.
- **Cops** ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) — spawn to the Heat
  target.

So "simulate the window, not the world" is *the* open-world principle, and traffic embodies it most visibly. A
player sees a living, endless city; the engine maintains only a small, churning slice of it around the camera. This
is how a 2005 game ([C58.1](../C58-Build-Pipeline/01-shipping-exe.md)) delivered a huge, populated open world on
modest hardware — the economy of the moving window, applied to every kind of content.

## RE implications

- **Anchor on** `AITrafficManager`, `TrafficLevel`, the spawners, and `AIVehicleTraffic`.
- **The RE workflow** — manager → density → spawners → brain.
- **Traffic completes the world's population** — cops + traffic + scenery, all via population managers, moving
  windows, and shared spawners.
- **Traffic embodies the open-world economy** — simulate the window, not the world.

---

### Key takeaways

- Traffic is anchored on **`AITrafficManager`**, **`TrafficLevel`**, the **spawners** (`SpawnTraffic`/
  `AIParkedCarSpawner`), and **`AIVehicleTraffic`**.
- The RE workflow: **manager → density → spawners → brain**.
- Traffic **completes the world's population** — cops, traffic, and scenery/smackables all use the same patterns
  (population manager, moving window, shared spawner, pooled actors).
- Traffic most clearly **embodies the open-world economy** — **"simulate the window, not the world"** — a whole
  city of cars, only your neighbourhood real.
- This principle (streaming, culling, traffic, cops) is **how a 2005 game delivered a huge populated open world** on
  modest hardware.

**Next:** [Chapter 62 — Physics Constraints, Joints & Trailers](../C62-Constraints-Joints/C62-Constraints-Joints.md):
bodies linked together.

**Sources:** `speed.exe` (verified: `AITrafficManager`/`TrafficManager` `0x00890F18`/29/948 B; `TrafficLevel`/
`TrafficSpeed`/`TrafficCar`; spawners `SpawnTraffic`/`AIParkedCarSpawner`/`Spawner`/`SpawnMode` and `SpawnCop`/
`SpawnCharacter`/`SpawnSmackable`; `AIVehicleTraffic` `0x00891C08`/195 on `AIGoalTraffic`; traffic sounds
`TrafficEngine`/`TrafficHorn`/`TrafficSkids`/`SFXCTL_3DTrafficPos`).
