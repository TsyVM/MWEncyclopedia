# C67.4 — Debug Cameras & Screens

> **The one-sentence version:** the debug cameras and screens are dev *shortcuts* — `DebugWatchCar`/
> `CDActionDebugWatchCar` (watch a specific car's behaviour), `DebugWorldCameraMover`/`DebugWorld` (free-fly the
> world), and `DebugVehicleSelection`/`DebugCarCustomize` (jump to any car/customization) — bypassing the
> player-facing flow to inspect and manipulate the game directly.

[← C67.3 — Debug overlays](03-debug-overlays.md) · [Chapter 67 hub](C67-Debug-Facilities.md) ·
[Next: C67.5 — Reading debug facilities in RE →](05-reading-debug.md)

---

## Debug cameras: watching the game

Debug output ([C67.2](02-debug-output.md)) and overlays ([C67.3](03-debug-overlays.md)) show you *state*; **debug
cameras** let developers *move the viewpoint* to inspect the game spatially. `speed.exe` names two:

- **`DebugWatchCar`** / **`CDActionDebugWatchCar`** — a director action ([C53.3](../C53-Cameras-Director/03-cinematic-director.md))
  that makes the camera *watch a specific car* — a cop, a racer, a traffic vehicle — so a developer could *follow*
  and observe that car's behaviour ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) — its AI
  driving, its physics, its pursuit logic ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **`DebugWorldCameraMover`** / **`DebugWorld`** — a *free-fly* debug camera: detach from the car and *move freely*
  through the world ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — to inspect geometry, check
  streaming ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)), or look at any
  part of the map.

So the debug cameras gave developers *free control of the viewpoint* — follow any car (`DebugWatchCar`) or fly
anywhere (`DebugWorld`), bypassing the normal camera system ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md))
that's locked to the player's car. `DebugWatchCar` as a *director action* ([C53.3](../C53-Cameras-Director/03-cinematic-director.md))
is especially neat — it plugs into the *same* camera director the game uses cinematically, so watching a debug car
reuses the cinematic camera machinery ([C67.3](03-debug-overlays.md)). These cameras are how developers *observed*
AI and physics ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) in the running game.

> ✅ *Verified:* `DebugWatchCar`, `CDActionDebugWatchCar`, `DebugWorldCameraMover`, `DebugWorld` are present in
> `speed.exe` — the debug cameras; `CDActionDebugWatchCar` is a director action
> ([C53.3](../C53-Cameras-Director/03-cinematic-director.md)).

## Debug screens: jumping the flow

Beyond cameras, `speed.exe` names **debug screens** — dev UIs that *bypass the normal game flow*
([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) to jump straight to a system:

- **`DebugVehicleSelection`** — a dev screen to *select any vehicle* directly, skipping the career/unlock flow
  ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) — pick any car for testing.
- **`DebugCarCustomize`** / **`DebugCarCustomizeScreen`** / **`DebugCarOption`** — dev screens for the customization
  system ([Chapter 56](../C56-Customization/C56-Customization.md)) — jump to any part, any option,
  without earning it.

So the debug screens are *shortcuts into the game's systems* — jump to any car (`DebugVehicleSelection`), any
customization (`DebugCarCustomize`), without playing through the career ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).
These let developers *test a specific thing* (a car, a part) immediately, without grinding to unlock it — essential
for iteration ([C58.4](../C58-Build-Pipeline/04-asset-pipeline.md)). That they're *debug* screens (named `Debug*`,
[C67.1](01-debug-in-shipped.md)) and *separate* from the player-facing screens ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md))
shows they were built *for development* — and shipped, dormant, in the retail flow.

> ✅ *Verified:* `DebugVehicleSelection`, `DebugCarCustomize`, `DebugCarCustomizeScreen`, `DebugCarOption` are
> present in `speed.exe` — the debug screens for vehicle selection
> ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) and customization
> ([Chapter 56](../C56-Customization/C56-Customization.md)).

## The pattern: bypass the player flow

The debug cameras and screens share a *pattern* — they **bypass the player-facing flow** to access systems directly
([C67.1](01-debug-in-shipped.md)):

- **The player flow is constrained.** A player must earn cars ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)),
  unlock parts ([Chapter 56](../C56-Customization/C56-Customization.md)), and view through the locked
  camera ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)) — the *designed experience*.
- **The debug flow is unconstrained.** A developer needs *immediate* access to *any* car, *any* part, *any*
  viewpoint — to test, tune, and debug without the constraints.
- **So the debug tooling is a parallel, unconstrained interface** — `DebugVehicleSelection` jumps past the unlock
  gate, `DebugWorld` flies past the camera lock — a dev's-eye view unbound by the player's rules.

So the debug cameras/screens are the *developer's* interface to the game — parallel to the player's, but
*unconstrained*. Where the player experiences the game *as designed* ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)),
the developer *inspects and manipulates* it directly. This is the essence of dev tooling
([C67.1](01-debug-in-shipped.md)): tools to *bypass the experience* and touch the *systems*. That these shipped
([C67.1](01-debug-in-shipped.md)) means the retail binary contains *both* interfaces — the player's constrained one
(active) and the developer's unconstrained one (dormant) — the game and its own making, in one executable.

## What they reveal about development

The debug cameras/screens *reveal how the game was developed* ([C67.5](05-reading-debug.md)):

- **The developers watched specific cars.** `DebugWatchCar` ([C53.3](../C53-Cameras-Director/03-cinematic-director.md))
  shows they debugged AI ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) and pursuit
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) by *following individual cars* — a targeted, per-entity
  debugging style.
- **They flew the world.** `DebugWorld` shows they inspected the world ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md))
  and streaming ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) by
  *free-flying* — checking geometry and residency across the map.
- **They jumped to systems.** `DebugVehicleSelection`/`DebugCarCustomize` show they tested cars
  ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) and customization
  ([Chapter 56](../C56-Customization/C56-Customization.md)) by *jumping straight to them* — iterating on
  one system at a time.

So the debug tooling *paints a picture of the development process*: developers followed individual cars to debug AI,
flew the world to check geometry and streaming, and jumped to systems to iterate on cars and parts. Each debug
facility is a *trace of a development need* — the tools exist because the developers had those needs. Reading them
([C67.5](05-reading-debug.md)) reconstructs *how Most Wanted was built and debugged*, from the tools its makers left
behind — the deepest sense in which the shipped game documents its own creation.

## RE implications

- **Debug cameras** (`DebugWatchCar`, `DebugWorldCameraMover`) move the viewpoint — watch a car
  ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) or free-fly the world.
- **Debug screens** (`DebugVehicleSelection`, `DebugCarCustomize`) jump the flow — any car
  ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)), any part
  ([Chapter 56](../C56-Customization/C56-Customization.md)).
- **The pattern is bypass** — a parallel, unconstrained developer interface alongside the player's constrained one.
- **They reveal the development** — following cars, flying the world, jumping to systems — traces of dev needs.

---

### Key takeaways

- **Debug cameras** — **`DebugWatchCar`/`CDActionDebugWatchCar`** (watch a specific car's AI/pursuit) and
  **`DebugWorldCameraMover`/`DebugWorld`** (free-fly the world) — gave developers **free control of the viewpoint**,
  bypassing the player-locked camera ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)).
- **Debug screens** — **`DebugVehicleSelection`** (any car) and **`DebugCarCustomize`/`DebugCarOption`** (any part) —
  **jump past the career/unlock flow** ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) to test a
  system immediately.
- The shared pattern is **bypass the player flow** — a **parallel, unconstrained developer interface** alongside the
  player's constrained one; the retail binary ships **both**.
- `DebugWatchCar` is a **director action** ([C53.3](../C53-Cameras-Director/03-cinematic-director.md)) — the debug
  camera **reuses the cinematic camera machinery**, another instance of the engine's uniformity.
- The debug cameras/screens **reveal the development process** — following cars to debug AI, flying to check
  streaming, jumping to iterate — each a **trace of a development need**, reconstructing how MW was built.

**Continue:** [C67.5 — Reading debug facilities in RE](05-reading-debug.md) · [Chapter 67 hub](C67-Debug-Facilities.md)
