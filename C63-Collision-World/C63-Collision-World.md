# Chapter 63 — The Collision World & Spatial Partitioning

> **Goal of this chapter:** decode the *spatial* side of collision (as opposed to the contact side of Chapter 43)
> — the `WCollisionPack` world collision geometry, the `AABB`/`Grid` broad-phase that finds collision candidates
> cheaply, the narrow-phase shape test, and the `CollisionCache` that makes it fast.

Chapter 43 decoded what happens *when* two things touch (contacts, reactions, damage); this chapter decodes how
the engine *finds* what might touch — efficiently, among thousands of collidable objects. It covers the world's
collision geometry (`WCollisionPack`, separate from the render meshes), the spatial partitioning
(`AABB` bounding boxes in a `Grid`) that culls the collision search, and the caching that keeps it cheap. It's the
*performance* side of collision — the reason a car can test against a whole city's geometry every frame.

> **Verified against the executable.** The collision world is named in `speed.exe`: **`WCollisionPack`** and
> **`WCollisionAssets`** (the world collision geometry), **`CollisionInstanceList`**/**`CollisionObjectList`** (the
> object lists), **`CollisionCache`** (caching), **`CollisionDetection`**/`CollisionDetectionWidget` (the detection
> + its debug overlay, [Chapter 67](../C67-Debug-Facilities/C67-Debug-Facilities.md)), `CollisionElement`,
> `CollisionEvent`. The spatial primitives are **`AABB`** (axis-aligned bounding box) and **`Grid`** (the spatial
> partition).
>
> **The on-disk collision data is verified byte-for-byte** ([C63.6](06-ondisk-collision-data.md)–[C63.9](09-smackables-emitters.md)):
> the three collision chunk loaders test their own IDs — `0x74B3A0` `cmp [eax],0x00034159` (terrain), `0x64AD80`
> `cmp [eax],0x0003B801` (`WCollisionPack` → manager `0x9B3890`), `0x6829D0` `cmp [eax],0x00034027` (smackables) —
> routed by the dispatcher at `0x45D600`; the terrain dequant constants `4.0 / 0.25 / 0.0625` sit at
> `0x890E98 / 0x8910F0 / 0x8B493C`. The format rebuilds bit-exact across the retail world's 720 stream sections.

---

## Deep-dive pages

**The runtime machine** ([C63.1](01-collision-world.md)–[C63.5](05-reading-collision-world.md)):

- [C63.1 — The collision world](01-collision-world.md): `WCollisionPack` and the object lists.
- [C63.2 — Broad-phase: AABB & Grid](02-broad-phase.md): finding candidates cheaply.
- [C63.3 — Narrow-phase](03-narrow-phase.md): the actual shape test.
- [C63.4 — CollisionCache & queries](04-collision-cache.md): caching, raycasts, spatial queries.
- [C63.5 — Reading the collision world in RE](05-reading-collision-world.md): navigating it.

**The on-disk data it loads** ([C63.6](06-ondisk-collision-data.md)–[C63.9](09-smackables-emitters.md)) — verified
byte-for-byte:

- [C63.6 — The on-disk collision data](06-ondisk-collision-data.md): the `0x45D600` dispatcher, the three chunk
  families, the 16-byte alignment invariant.
- [C63.7 — Terrain collision mesh](07-terrain-collision.md): the ground-height triangle soup (`0x00034159`,
  quantized `/4,/4,/16`).
- [C63.8 — Wall & object collision](08-wcollisionpack.md): `WCollisionPack` as a CARP article container
  (`0x0003B801`).
- [C63.9 — Smackables & FX placements](09-smackables-emitters.md): knock-down props (`0x00034027`) + emitters
  (`0x0003BC00`).

---

## 63.1 The collision world

The world has a **collision geometry** — `WCollisionPack` ([C63.1](01-collision-world.md)) — *separate* from the
render meshes ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)). Collision geometry is *simpler* (fewer
polygons, just the shapes to collide against) so the physics can test against it cheaply. The
`CollisionInstanceList`/`CollisionObjectList` ([C43.1](../C43-Collision-Contacts/01-detection.md)) hold the
collidable objects — the static world and the dynamic bodies — that a body tests against each frame.

## 63.2 Broad-phase: AABB & Grid

Testing a body against *every* collidable object would be far too slow, so collision uses a **broad-phase**
([C63.2](02-broad-phase.md)): each object has an **`AABB`** (axis-aligned bounding box) — a cheap rectangular
bound — and the objects are organised in a **`Grid`** (spatial partition). To find what a body *might* hit, the
engine checks only the objects in the body's grid cells whose AABBs overlap — a tiny fraction of the world. This
culls the search from "everything" to "the few nearby" before any expensive test.

## 63.3 Narrow-phase

The broad-phase gives *candidates*; the **narrow-phase** ([C63.3](03-narrow-phase.md)) does the *actual* shape
test on each — the precise geometry check that produces the contact ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)).
Only the few candidates that survived the broad-phase ([C63.2](02-broad-phase.md)) get this expensive test, so the
per-frame cost is bounded. The `CollisionElement` is the shape primitive tested. Broad-phase (cheap, many) +
narrow-phase (expensive, few) is the standard two-stage collision pipeline.

## 63.4 CollisionCache & queries

The **`CollisionCache`** ([C63.4](04-collision-cache.md)) makes collision fast by *reusing* results — contacts and
spatial lookups persist frame-to-frame where the geometry hasn't changed, avoiding recomputation. The collision
world also serves **queries** beyond body-body contact: **raycasts** (the wheels' ground tests,
[C43.1](../C43-Collision-Contacts/01-detection.md); the AI's line-of-sight and avoidance,
[C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)) — asking "what's along this ray?" against the same
spatial structure ([C63.2](02-broad-phase.md)).

## 63.5 The on-disk collision data

The runtime machine above is *fed* by collision data baked into every stream section
([C63.6](06-ondisk-collision-data.md)) — decoded byte-for-byte against the retail world. Three chunk families,
routed by the dispatcher at `0x45D600`, carry it: the **terrain collision mesh** (`0x00034159`,
[C63.7](07-terrain-collision.md)) — a quantized triangle *soup* that answers the wheels' ground-height query; the
**`WCollisionPack`** (`0x0003B801`, [C63.8](08-wcollisionpack.md)) — a CARP article container of corridor-edge
geometry, the walls that stop the car; and the **smackable spawners** (`0x00034027`,
[C63.9](09-smackables-emitters.md)) — the physics records for knock-down props, tracked by a per-section 2048-bit
"already smashed" mask. Every record-bearing chunk obeys the same 16-byte alignment invariant `(ptr + 0x17) & ~0xF`,
which is both how the loader finds its records and the reason collision edits must be *size-neutral*. This is the
data side of the collision world — the floor, the walls, and the breakables, as actual bytes.

---

### Key takeaways

- The world has a **collision geometry** (`WCollisionPack`) *separate from* the render meshes — simpler, cheaper to
  test against.
- Collision uses a **two-stage pipeline**: **broad-phase** (`AABB` bounds in a `Grid` — find the few nearby
  candidates cheaply) then **narrow-phase** (the precise shape test on the few candidates).
- The **`CollisionCache`** reuses results frame-to-frame where geometry is unchanged — keeping collision cheap.
- The collision world serves **queries** too — **raycasts** for wheel ground-tests and AI line-of-sight — against
  the same spatial structure.
- This is the **performance side** of collision — how a body tests against a whole city's geometry every frame
  (the *contact* side is [Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)).
- The runtime machine is **fed by on-disk collision data** ([C63.6](06-ondisk-collision-data.md)–[C63.9](09-smackables-emitters.md))
  — the **terrain mesh** (`0x00034159`, the floor), **`WCollisionPack`** (`0x0003B801`, the walls), and
  **smackables** (`0x00034027`, the breakables) — decoded **byte-for-byte** and dispatched through `0x45D600`.

**Next:** [Chapter 64 — World Update: Bodies, Effects & the Active Lists](../C64-World-Update/C64-World-Update.md):
the tick that advances everything.
