# C63.1 — The Collision World

> **The one-sentence version:** the world has a `WCollisionPack` collision geometry — separate from and simpler
> than the render meshes — held in `CollisionInstanceList`/`CollisionObjectList`, that bodies test against each
> frame.

[← Chapter 63 hub](C63-Collision-World.md) · [Next: C63.2 — Broad-phase: AABB & Grid →](02-broad-phase.md)

---

## Collision geometry ≠ render geometry

A key fact: the world's **collision geometry is separate** from its render geometry
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)). The verified **`WCollisionPack`** (and
`WCollisionAssets`) is the *collision* representation of the world — the shapes bodies collide against — distinct
from the detailed meshes the renderer draws ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)):

- **Render geometry** — high-polygon, textured, detailed (a building with windows, trim, detail).
- **Collision geometry** — low-polygon, simplified (the building as a few flat walls) — just enough to collide
  against correctly.

Why separate them? Because *colliding* against high-detail meshes would be far too slow — testing a car against
every window-frame polygon is wasteful. A simplified collision mesh (a box for the building) gives correct collision
at a fraction of the cost. So the world ships *two* geometries: the detailed one to *draw*
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), the simplified one to *collide*
(`WCollisionPack`). This render/collision split is universal in game engines, and MW's is `WCollisionPack`.

> ✅ *Verified:* `WCollisionPack` and `WCollisionAssets` are present in `speed.exe` — the world collision geometry;
> `CollisionInstanceList`/`CollisionObjectList` ([C43.1](../C43-Collision-Contacts/01-detection.md)) hold the
> collidable objects.

## The object lists

The collidable objects are held in two verified lists ([C43.1](../C43-Collision-Contacts/01-detection.md)):

- **`CollisionObjectList`** — the *objects* that can collide (the world's collision shapes, the dynamic bodies).
- **`CollisionInstanceList`** — the *instances* — placed occurrences of collision shapes in the world (a shared
  building shape instanced at many locations, [Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)).

So the collision world is a *list of instances* of collision shapes — the static world (buildings, walls, ground,
from `WCollisionPack`) plus the dynamic bodies (cars, [Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)).
A body's collision detection ([C43.1](../C43-Collision-Contacts/01-detection.md)) tests against these lists — but
*not* naively against all of them ([C63.2](02-broad-phase.md)); the spatial partition
([C63.2](02-broad-phase.md)) narrows the search first. The instance/object split
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) means one collision *shape* is shared across many
*instances* — the same memory economy as the render scenery
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)).

## Static vs. dynamic collidables

The collision world holds two kinds of collidable ([above](#the-object-lists)):

- **Static** — the world geometry (`WCollisionPack`) — buildings, walls, ground, barriers
  ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)). It never moves, so its spatial organisation
  ([C63.2](02-broad-phase.md)) is *baked* — computed once at load.
- **Dynamic** — the bodies ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) — cars, smackables
  ([C43.5](../C43-Collision-Contacts/05-smackables.md)), debris. They move, so their spatial organisation must be
  *updated* each frame.

This static/dynamic split matters for performance ([C63.2](02-broad-phase.md)): the static world can be
pre-organised (a fixed grid, [C63.2](02-broad-phase.md)) since it never changes, while the dynamic bodies are
re-inserted each frame as they move. Most collision is a car (dynamic) against the world (static) — the common
case, optimised by the static world's baked spatial structure. Recognising this split
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) explains the collision world's organisation: a fixed,
baked static world plus a churning set of dynamic bodies.

> 🟡 *Reasoned:* the static/dynamic split and the baked-vs-updated spatial organisation are the standard collision-world
> design, consistent with the verified `WCollisionPack`/`CollisionInstanceList` and the streaming model; the exact
> baking is deeper RE. The collision geometry and lists are verified.

## Why a collision world

Having a dedicated collision world ([above](#collision-geometry--render-geometry)) — separate geometry, organised
lists — is essential:

- **Cheap collision** — the simplified geometry ([above](#collision-geometry--render-geometry)) makes testing fast;
  a car against a box-building, not a detailed mesh.
- **Organised search** — the lists + spatial partition ([C63.2](02-broad-phase.md)) let a body find nearby
  collidables without scanning the world.
- **Shared with queries** — the same collision world serves raycasts ([C63.4](04-collision-cache.md)) for wheels
  and AI — one spatial structure, many uses.

So the collision world is the *spatial substrate* of the physics — the organised, simplified representation of
"what's solid and where" that every collision and query runs against. It's the counterpart to the render world
([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) — one for drawing, one for colliding — and its
separateness is the key to affordable collision. Reading it is understanding *how the physics knows the world's
shape*.

## RE implications

- **Collision geometry (`WCollisionPack`) is separate from render geometry** — simplified, cheaper to test.
- **The object lists** (`CollisionInstanceList`/`ObjectList`) hold the collidables — instances of shared shapes.
- **Static vs. dynamic** — the world (baked spatial structure) + the bodies (updated each frame).
- **A dedicated collision world** gives cheap collision, organised search, and shared queries.

---

### Key takeaways

- The world's **collision geometry (`WCollisionPack`) is separate from and simpler than the render meshes** — a
  box-building to collide against, not a detailed one to draw — making collision **cheap**.
- The collidables are held in **`CollisionObjectList`/`CollisionInstanceList`** — instances of shared collision
  shapes (the memory economy of instancing).
- The collision world splits **static** (the baked world geometry, never moving) from **dynamic** (the bodies,
  re-inserted each frame) — most collision is a car (dynamic) vs. the world (static).
- A dedicated collision world gives **cheap collision** (simplified geometry), **organised search** (lists +
  spatial partition, [C63.2](02-broad-phase.md)), and **shared queries** (raycasts).
- It's the **spatial substrate** of the physics — the counterpart to the render world, one for drawing, one for
  colliding.

**Continue:** [C63.2 — Broad-phase: AABB & Grid](02-broad-phase.md) · [Chapter 63 hub](C63-Collision-World.md)
