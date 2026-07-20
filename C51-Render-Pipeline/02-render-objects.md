# C51.2 — Render Objects & Pools

> **The one-sentence version:** the renderer draws pooled `RenderObject`s — each a `RenderInfo` (what) plus a
> `RenderMethod` (how) — allocated from pre-sized slot pools (`RenderObjectSlotPool`, `RenderEPolySlotPool`), fed
> from the simulation through the `RenderConn` connector.

[← C51.1 — The Direct3D 9 foundation](01-d3d9-foundation.md) · [Chapter 51 hub](C51-Render-Pipeline.md) ·
[Next: C51.3 — The effect & shader system →](03-effect-system.md)

---

## The render object

The renderer doesn't draw the game's simulation objects directly — it draws **render objects**, a rendering-side
representation of what's visible. The verified classes:

- **`RenderObject`** — one drawable thing (a car, a building, a prop) as the renderer sees it.
- **`RenderInfo`** — *what* to draw: the geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), the
  transform, the material binding.
- **`RenderMethod`** — *how* to draw it: which shader path / effect ([C51.3](03-effect-system.md)), which render
  state.

So a render object separates the *data* (RenderInfo — mesh, position, material) from the *technique* (RenderMethod
— shader, state). This lets the same geometry render different ways (a car in full detail vs. a cheap reflection),
and different geometry share a technique (all opaque scenery through one RenderMethod).

> ✅ *Verified:* `RenderObject`, `RenderInfo`, `RenderMethod`, `RenderConn`/`eRenderConn`, `RenderAssets`, and
> `RenderingCar` are present as strings in `speed.exe`; the render pools `RenderObjectSlotPool` and
> `RenderEPolySlotPool` (+`Overflow`) are pool allocators.

## The slot pools

Render objects are **pooled** ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — the verified
`RenderObjectSlotPool` and `RenderEPolySlotPool` are pre-sized allocators:

- **`RenderObjectSlotPool`** — the pool of render objects; a fixed number of slots, reused frame to frame.
- **`RenderEPolySlotPool`** (+ `RenderEPolySlotPoolOverflow`) — the pool of "EPoly" render primitives, with an
  *overflow* pool for when the main one fills.

Pooling the render objects means **no per-frame heap allocation** for drawing — the renderer draws from fixed
slots, so the frame's rendering has a bounded, predictable memory cost and no allocation stalls
([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)). The presence of an *overflow* pool
(`…Overflow`) is a telling detail: the main pool is sized for the common case, and a spillover pool catches the
occasional heavy frame — a pragmatic "size for typical, handle the peak" design.

## RenderConn: sim to renderer

The **`RenderConn`/`eRenderConn`** connector ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) is the bridge
from a *simulated* object to its *render* object. A car ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
has a render connector that, each frame, hands its state (the transform from `IntegrateMotion`
[C39.4](../C39-Vehicle-Simulation/04-integrate.md), its damage state
[Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md), its wheel angles) to the renderer's
`RenderObject`. This is the *one-way* connector boundary ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) in
action: the sim publishes state, the renderer reads it, and the renderer *can't* perturb the sim. `RenderingCar` is
the render-side view of a car — the drawable the connector feeds.

So the data flow is: sim object → `RenderConn` → `RenderObject` (`RenderInfo` + `RenderMethod`) → draw. The
connector decouples the two sides, so the renderer and simulation run on their own terms
([C39.5](../C39-Vehicle-Simulation/05-connectors.md)), meeting only through the published render state.

> 🟡 *Reasoned:* the RenderConn-feeds-RenderObject data flow is the connector model
> ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) applied to rendering, consistent with the verified
> `RenderConn`/`RenderingCar` classes; the exact per-frame update is deeper RE. The classes and pools are verified.

## Why a render-object layer

Interposing a render-object layer between the simulation and the D3D9 device ([C51.1](01-d3d9-foundation.md)) buys
the renderer several things:

- **Decoupling** — the sim doesn't know about D3D9; it publishes state to a `RenderConn`, and the render layer
  translates to draws. The graphics API can change without touching the sim.
- **Batching and sorting** — with all render objects gathered, the renderer can sort by material/state to minimise
  expensive GPU state changes, and batch similar draws. Drawing straight from sim objects couldn't do this.
- **LOD and culling** — the render layer decides detail level and visibility
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) per render object, independent of the sim.
- **Pooling** ([above](#the-slot-pools)) — a fixed, bounded rendering cost.

So the render-object model is the renderer's *organisation* of the frame: gather the visible objects (pooled),
sort/batch them, and draw them through the device. It's the layer that turns "here's the world state" into "here's
an efficient sequence of GPU commands" ([C51.5](05-render-frame.md)).

## RE implications

- **Render objects** — `RenderObject` = `RenderInfo` (what) + `RenderMethod` (how) — separate data from technique.
- **Pooled** — `RenderObjectSlotPool`/`RenderEPolySlotPool` (+Overflow) — no per-frame heap churn, bounded cost.
- **`RenderConn`** bridges sim → renderer, one-way ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)); `RenderingCar` is the car's drawable.
- **The render layer** decouples from D3D9, enables batching/sorting/LOD, and bounds memory.

---

### Key takeaways

- The renderer draws **render objects** — `RenderObject` = **`RenderInfo`** (what: geometry, transform, material) +
  **`RenderMethod`** (how: shader path, state).
- They're **pooled** — `RenderObjectSlotPool` and `RenderEPolySlotPool` (+`Overflow`) — so rendering has **no
  per-frame heap allocation** and a bounded cost (the overflow pool catches heavy frames).
- The **`RenderConn` connector** feeds sim state (transform, damage, wheels) to the render object — the one-way
  boundary, so the renderer reads but can't perturb the sim.
- The **render-object layer** decouples the sim from D3D9, and enables **batching, sorting, and LOD** the sim
  couldn't do.
- Data flows **sim → `RenderConn` → `RenderObject` → draw** — turning world state into efficient GPU commands.

**Continue:** [C51.3 — The effect & shader system](03-effect-system.md) · [Chapter 51 hub](C51-Render-Pipeline.md)
