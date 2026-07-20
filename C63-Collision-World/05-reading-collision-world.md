# C63.5 — Reading the Collision World in RE

> **The one-sentence version:** navigate the collision world by `WCollisionPack`, the object lists, the `AABB`/
> `Grid` broad-phase, `CollisionElement`, and `CollisionCache` — reading collision as a two-stage, cached,
> spatially-partitioned service.

[← C63.4 — CollisionCache & queries](04-collision-cache.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: Chapter 64 — World Update: Bodies, Effects & the Active Lists →](../C64-World-Update/C64-World-Update.md)

---

## Anchors for collision-world RE

The collision world is anchored on verified strings:

- **The geometry** — `WCollisionPack`, `WCollisionAssets` ([C63.1](01-collision-world.md)).
- **The lists** — `CollisionInstanceList`, `CollisionObjectList` ([C63.1](01-collision-world.md)).
- **The broad-phase** — `AABB`, `Grid` ([C63.2](02-broad-phase.md)).
- **The narrow-phase** — `CollisionElement`, `CollisionDetection` ([C63.3](03-narrow-phase.md)).
- **The cache** — `CollisionCache` ([C63.4](04-collision-cache.md)).

From these, the collision world is navigable: geometry, lists, broad-phase, narrow-phase, and cache.

## The RE workflow

Reading the collision world:

1. **Find the geometry** — `WCollisionPack` ([C63.1](01-collision-world.md)); the world's collision shapes.
2. **Trace the broad-phase** — `AABB`/`Grid` ([C63.2](02-broad-phase.md)); the spatial cull.
3. **Trace the narrow-phase** — `CollisionElement`/`CollisionDetection` ([C63.3](03-narrow-phase.md)); the precise
   test.
4. **Find the cache and queries** — `CollisionCache` and the raycasts ([C63.4](04-collision-cache.md)).

The output is the full collision-world picture: geometry, the two-stage pipeline, and the cache/queries.

## The spatial optimisation is the lesson

The organising lesson of this chapter is the *layered optimisation* that makes collision affordable
([C63.2](02-broad-phase.md)–[C63.4](04-collision-cache.md)):

```
simplified geometry (C63.1)   — cheap shapes, not detailed meshes
   + spatial partition (Grid, C63.2)  — only nearby objects
      + AABB pre-test (C63.2)          — cheap rule-outs
         + narrow-phase on few (C63.3) — precise only where needed
            + cache (C63.4)            — reuse unchanged results
```

Each layer *multiplies* the savings — simplified geometry × spatial culling × AABB rule-outs × few precise tests ×
temporal caching. Together they turn an impossible O(N²) precise-test-everything into an affordable near-O(1)-per-body
service. This *stacked optimisation* is the deepest lesson of the collision world: real-time collision isn't *one*
trick but a *stack* of them, each cutting the cost further. Reading the collision world is reading how careful
engineering ([C58.2](../C58-Build-Pipeline/02-eagl-engine.md)) makes a hard problem (collision against a whole city)
cheap enough to run every frame.

## Collision completes the physics substrate

With the collision world decoded, the *physics substrate* is complete:

- **The bodies** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) — what collides.
- **The contacts** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) — the response to a
  collision.
- **The collision world** (this chapter) — *finding* the collisions efficiently.
- **Constraints** ([Chapter 62](../C62-Constraints-Joints/C62-Constraints-Joints.md)) — linked bodies.

So the physics spans the *bodies*, their *contacts*, the *world* they collide in, and their *linkages* — the
complete collision/dynamics system. The collision world is the *spatial engine* underneath it all: the grid, the
AABBs, the narrow-phase, the cache that let every body test against the world every frame. Reading it completes the
picture of *how the physics knows what's touching what* — the spatial foundation the contacts
([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) and dynamics
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) build on. The next chapter
([64](../C64-World-Update/C64-World-Update.md)) is the *tick* that drives it all — the world update.

## RE implications

- **Anchor on** `WCollisionPack`, the object lists, `AABB`/`Grid`, `CollisionElement`, and `CollisionCache`.
- **The RE workflow** — geometry → broad-phase → narrow-phase → cache/queries.
- **The spatial optimisation is stacked** — simplified geometry × grid × AABB × few precise × cache.
- **Collision completes the physics substrate** — bodies, contacts, the world, and constraints.

---

### Key takeaways

- The collision world is anchored on **`WCollisionPack`**, the **object lists**, the **`AABB`/`Grid`** broad-phase,
  **`CollisionElement`** narrow-phase, and **`CollisionCache`**.
- The RE workflow: **geometry → broad-phase → narrow-phase → cache/queries**.
- The lesson is **stacked optimisation** — simplified geometry × spatial grid × AABB rule-outs × few precise tests
  × temporal caching — each layer multiplying the savings, turning O(N²) into near-O(1)-per-body.
- Real-time collision isn't **one trick but a stack** — careful engineering making collision against a whole city
  cheap enough to run every frame.
- Collision **completes the physics substrate** — bodies (Ch 41), contacts (Ch 43), the **collision world** (this
  chapter), and constraints (Ch 62) — the spatial foundation of the whole physics.

**Next:** [Chapter 64 — World Update: Bodies, Effects & the Active Lists](../C64-World-Update/C64-World-Update.md):
the tick that advances everything.

**Sources:** `speed.exe` (verified: `WCollisionPack`/`WCollisionAssets`; `CollisionInstanceList`/`CollisionObjectList`;
`AABB`/`Grid`; `CollisionElement`/`CollisionDetection`/`CollisionDetectionWidget`; `CollisionCache`;
`CollisionEvent`).
