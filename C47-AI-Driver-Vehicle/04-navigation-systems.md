# C47.4 — Navigation & the Named Systems

> **The one-sentence version:** the AI navigates via `PathFinder` (an A\* search over the road graph, verified by
> its `AStarNodeSlotPool`/`AStarSearchSlotPool`), `Gps` (the player's route), and `AvoidableManager` (hazards) —
> all over `WRoadNetwork`, one of five named singleton systems (`World`, `VehicleSystem`, `WRoadNetwork`,
> `CameraAI`, `SceneryModel`) that are the roots of the object graph.

[← C47.3 — The managers](03-managers.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md) ·
[Next: C47.5 — Reading the driver brain in RE →](05-reading-ai-brain.md)

---

## PathFinder: A* over the road graph

For an AI to drive *somewhere* — a cop to intercept you, a racer to follow the route — it must find a path through
the roads. That's **`PathFinder`** (verified vtable `0x008B5C50`, 16 methods, 84 B): the **A\* search engine** over
the road graph. Its identity is proven by its constructor's allocations:

- **`AStarNodeSlotPool`** — a pre-sized pool of A\* search nodes.
- **`AStarSearchSlotPool`** — a pre-sized pool of A\* searches.

These verified pool names confirm the routing is **classic A\*** with **pre-sized node/open-set pools** — no
per-query allocation ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)), so searches are cheap and
bounded (a fixed memory budget, no allocation stalls mid-frame). `PathFinder` serves everyone: cop intercepts, GPS
routes, and traffic routing all call the same A\* over the same graph.

> ✅ *Verified:* `PathFinder` is a real vtable at `0x008B5C50` (16 methods); its constructor allocates
> `AStarNodeSlotPool` and `AStarSearchSlotPool` (both verified strings) — confirming pooled A\* routing. It
> references `AIVehicle` (its caller).

## WRoadNetwork: the shared graph

All navigation runs over **`WRoadNetwork`** — the runtime road graph, the in-memory form of the CARP road network
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)): nodes (`RNnd`), segments (`RNsg`), and the cost
grids that `PathFinder` searches. It's the **single source of truth for "where are the roads and how do they
connect,"** shared by:

- **Traffic** ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) — ambient cars follow the graph's
  lanes.
- **Cops** — pursuit and interception route over it ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **Navigation** — `Gps` ([below](#gps-the-players-route)) routes the player over it.

That every AI agrees on *one* graph is what makes a city-wide pursuit coherent
([C47.3](03-managers.md)): the cops route to you, the traffic flows, and your GPS all reference the same roads. The
road network ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) is thus the substrate of all AI
movement — decoded from the CARP `0x0003B800` payload
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).

## Gps and AvoidableManager

Two more navigation services complete the picture:

- **`Gps`** (verified vtable `0x008911CC`, 16 methods, 888 B) — the **player's navigation**: it owns the route
  from your position to the current objective and renders the on-road **direction arrow**
  ([Chapter 25](../C27-FrontEnd-Shell-UI/04-hud.md)). Its 888 bytes are mostly the route-node buffer; only 16
  methods, because it's a *data service* (queried by the HUD and camera), routing over `WRoadNetwork` via
  `PathFinder`.
- **`AvoidableManager`** (verified vtable `0x00890CDC`, 17 methods, 76 B) — the **hazard registry**: it tracks
  "avoidables" (other cars, hazards, deployed spike strips
  [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md), debris) and answers avoidance
  queries for the driving actions ([Chapter 46](../C46-AI-Goals-Actions/05-action-menu.md)). Small (76 B) because
  it's an *index*, not a simulator — the shared service that keeps racers and cops from ploughing into known
  obstacles.

So navigation is layered: `WRoadNetwork` (the graph), `PathFinder` (search it), `Gps` (the player's route), and
`AvoidableManager` (what to steer around). Every AITogether they answer "where can I go, how do I get there, and
what must I avoid."

> ✅ *Verified:* `Gps` (`0x008911CC`, 16 methods) and `AvoidableManager` (`0x00890CDC`, 17 methods) are confirmed
> vtables; `Gps` references `GPS`/`MARKER_DIRECTION_AID`/`WorldUpdate`.

## The five named systems

At the very top are five **named singleton systems** — registered *by name* (not the factory) on list `0x00988DFC`
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)), because there is *exactly one* of
each. They are the roots of the object graph:

| System | Owns |
|---|---|
| `World` | the world root — sections, the active-body/effect lists, the global tick |
| `VehicleSystem` | the vehicle pool/factory — builds every `PVehicle` from its vault recipe |
| `WRoadNetwork` | the road graph ([above](#wroadnetwork-the-shared-graph)) |
| `CameraAI` | the automatic gameplay camera director |
| `SceneryModel` | the scenery-instance registry ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) |

These register *by name* rather than being mass-produced from a hash ([C33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md))
because they're *the* world, *the* vehicle pool, *the* road graph — created once at load, not on demand. The engine
keeps their human-readable name in the registry node (for the console/debugger) instead of an allocator pointer —
and that naming *is* the structural proof they're top-level systems. `World` is the root everything hangs off;
`VehicleSystem` builds the cars; `WRoadNetwork` is the map they drive on.

> ✅ *Verified:* the five named systems `World`, `VehicleSystem`, `WRoadNetwork`, `CameraAI`, `SceneryModel` are
> present as strings in `speed.exe`, registered on the named-system list `0x00988DFC`
> ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).

## RE implications

- **`PathFinder`** is pooled **A\*** over the road graph (verified `AStarNodeSlotPool`/`AStarSearchSlotPool`) —
  serving cops, GPS, and traffic alike.
- **`WRoadNetwork`** is the **shared road graph** (from CARP) — one source of truth, why a city-wide pursuit is
  coherent.
- **`Gps`** (player route) and **`AvoidableManager`** (hazard index) complete navigation.
- **Five named singleton systems** (`World`/`VehicleSystem`/`WRoadNetwork`/`CameraAI`/`SceneryModel`) on
  `0x00988DFC` are the object-graph roots.

---

### Key takeaways

- The AI navigates via **`PathFinder`** — pooled **A\*** over the road graph, verified by its
  `AStarNodeSlotPool`/`AStarSearchSlotPool` (no per-query allocation, cheap bounded searches).
- All navigation runs over **`WRoadNetwork`** — the shared runtime road graph (from CARP,
  [Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) — the single source of truth that makes
  city-wide pursuit coherent.
- **`Gps`** (the player's route + direction arrow) and **`AvoidableManager`** (the hazard index) complete the
  navigation layer.
- Five **named singleton systems** — `World`, `VehicleSystem`, `WRoadNetwork`, `CameraAI`, `SceneryModel` — are
  registered by name on `0x00988DFC`, the **roots of the object graph** (one each, created at load).
- Navigation is **layered**: the graph (`WRoadNetwork`), searching it (`PathFinder`), the player's route (`Gps`),
  and avoidance (`AvoidableManager`).

**Continue:** [C47.5 — Reading the driver brain in RE](05-reading-ai-brain.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md)
