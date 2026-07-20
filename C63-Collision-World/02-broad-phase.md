# C63.2 — Broad-Phase: AABB & Grid

> **The one-sentence version:** to avoid testing a body against every object, collision uses a broad-phase — each
> object has a cheap `AABB` bound, organised in a spatial `Grid`, so a body checks only the few objects in its
> nearby cells whose AABBs overlap.

[← C63.1 — The collision world](01-collision-world.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: C63.3 — Narrow-phase →](03-narrow-phase.md)

---

## The problem: too many objects

A city has *thousands* of collidable objects ([C63.1](01-collision-world.md)) — buildings, walls, cars, props.
Testing a body against *all* of them each frame would be O(N) per body — catastrophically slow with a busy world.
The solution is a **broad-phase**: a cheap pre-filter that finds the *few* objects a body might collide with,
before any expensive shape test ([C63.3](03-narrow-phase.md)). MW's broad-phase uses two verified primitives: the
**`AABB`** and the **`Grid`**.

> ✅ *Verified:* `AABB` (axis-aligned bounding box) and `Grid` (spatial partition) are present in `speed.exe` — the
> broad-phase primitives.

## AABB: the cheap bound

An **`AABB`** — Axis-Aligned Bounding Box — is a *cheap* proxy for an object's shape: a box, aligned to the world
axes, that fully contains the object. It's the fastest possible overlap test:

- **Overlap in O(1)** — two AABBs overlap iff their ranges overlap on all three axes — six comparisons, no geometry.
- **A conservative bound** — the AABB contains the object, so *if two AABBs don't overlap, the objects can't
  collide* (definitely). If they *do* overlap, the objects *might* collide (test precisely,
  [C63.3](03-narrow-phase.md)).

So the AABB is the *first filter*: comparing bounding boxes is far cheaper than comparing shapes, and a non-overlap
*rules out* a collision instantly. Every collidable ([C63.1](01-collision-world.md)) has an AABB, and the
broad-phase works in AABBs, deferring the expensive shape test ([C63.3](03-narrow-phase.md)) to the few pairs whose
AABBs overlap. The AABB is the workhorse of cheap collision culling.

## Grid: the spatial partition

Even AABB-vs-AABB for *all* pairs is O(N²) — still too slow. The **`Grid`** solves this: it partitions space into
cells, and each object is placed in the cell(s) its AABB covers. Then a body only checks objects in its *own*
cells:

```
the world is divided into a grid of cells
each object is inserted into the cell(s) its AABB overlaps
to find a body's collision candidates:
   look up the body's cell(s)
   → check only the objects in those cells (a handful)
   → AABB-test those, then shape-test the overlaps (C63.3)
```

So the grid reduces the search from *all objects* to *objects in nearby cells* — a tiny fraction. A car in
downtown checks only the downtown cells' objects, not the whole city. This is the spatial-partitioning payoff:
O(N²) becomes roughly O(N) (each object checks only its local neighbourhood). The `Grid`
([above](#the-problem-too-many-objects)) is what makes collision *scale* — a whole city of objects, but each body
tests against only its local cells. The static world is baked into the grid at load
([C63.1](01-collision-world.md)); dynamic bodies are re-inserted each frame as they move.

> 🟡 *Reasoned:* the grid-partition + AABB broad-phase is the standard collision-culling approach, consistent with
> the verified `AABB`/`Grid` primitives and the collision world ([C63.1](01-collision-world.md)); the exact grid
> cell size and insertion are deeper RE. The `AABB` and `Grid` primitives are verified.

## Broad-phase then narrow-phase

The broad-phase ([above](#grid-the-spatial-partition)) is the *first* of two collision stages
([C63.3](03-narrow-phase.md)):

```
broad-phase (cheap, culls many):
   grid → nearby objects → AABB test → candidate pairs
      ↓
narrow-phase (expensive, few, C63.3):
   precise shape test on each candidate → actual contacts (Ch 43)
```

So the pipeline is *cull cheaply, then test precisely*: the broad-phase (grid + AABB) reduces thousands of objects
to a handful of candidate pairs, and the narrow-phase ([C63.3](03-narrow-phase.md)) does the expensive precise
geometry test on just those. This two-stage structure is *the* standard collision architecture, and it's why
collision is affordable: the expensive test runs only on the few pairs the cheap filter couldn't rule out. Without
the broad-phase, collision would be impossibly slow; with it, a body tests against a whole city each frame for the
cost of a few precise tests. The `Grid` and `AABB` ([above](#aabb-the-cheap-bound)) are the enabling technology.

## RE implications

- **Broad-phase** pre-filters collision — finding the few candidates before the expensive shape test.
- **`AABB`** is the cheap bound — O(1) overlap; a non-overlap rules out collision instantly.
- **`Grid`** is the spatial partition — a body checks only its nearby cells' objects (O(N²) → ~O(N)).
- **Broad then narrow** — cull cheaply (grid + AABB), then test precisely ([C63.3](03-narrow-phase.md)) — the
  standard architecture.

---

### Key takeaways

- Collision uses a **broad-phase** to avoid testing a body against every object — a cheap pre-filter finding the
  few candidates.
- **`AABB`** (axis-aligned bounding box) is the **cheap bound** — O(1) overlap (six comparisons); a **non-overlap
  rules out collision instantly**.
- **`Grid`** is the **spatial partition** — objects are placed in cells, and a body checks only its **nearby cells'**
  objects, reducing O(N²) to roughly O(N).
- The pipeline is **broad-phase (cull cheaply) then narrow-phase (test precisely,
  [C63.3](03-narrow-phase.md))** — the standard collision architecture.
- The grid + AABB are what make collision **scale** — a body tests against a **whole city** each frame for the cost
  of a **few precise tests**.

**Continue:** [C63.3 — Narrow-phase](03-narrow-phase.md) · [Chapter 63 hub](C63-Collision-World.md)
