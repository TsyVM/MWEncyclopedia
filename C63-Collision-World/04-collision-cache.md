# C63.4 — CollisionCache & Queries

> **The one-sentence version:** the `CollisionCache` reuses collision results frame-to-frame where geometry hasn't
> changed, and the collision world serves queries beyond body-body contact — raycasts for the wheels' ground tests
> and the AI's line-of-sight and avoidance.

[← C63.3 — Narrow-phase](03-narrow-phase.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: C63.5 — Reading the collision world in RE →](05-reading-collision-world.md)

---

## CollisionCache: reuse results

Even with the two-stage pipeline ([C63.2](02-broad-phase.md)–[C63.3](03-narrow-phase.md)), recomputing collision
from scratch every frame is wasteful — much doesn't change frame-to-frame. The verified **`CollisionCache`**
avoids the waste by *reusing* results:

- **Persistent contacts** — a car resting against a wall has the *same* contact frame after frame; the cache keeps
  it rather than re-detecting it ([C43.2](../C43-Collision-Contacts/02-contact-records.md)).
- **Cached spatial lookups** — a body's grid cell ([C63.2](02-broad-phase.md)) and nearby candidates change slowly;
  cache them and update incrementally rather than re-querying wholesale.
- **Coherence exploitation** — frame-to-frame, the world is *coherent* (things move a little, not teleport), so
  last frame's collision state is a good starting point for this frame's — the cache leverages this.

So the `CollisionCache` is a *temporal* optimisation — it exploits that collision changes *gradually*, reusing
last frame's work where valid. This complements the *spatial* optimisation (the broad-phase,
[C63.2](02-broad-phase.md)): the broad-phase avoids testing far things; the cache avoids re-testing unchanged
things. Together they make collision cheap in *both* dimensions — space (only nearby) and time (only changed). This
is the standard temporal-coherence optimisation, and `CollisionCache` is MW's.

> ✅ *Verified:* `CollisionCache` is present in `speed.exe` — the collision result cache. It complements the
> `AABB`/`Grid` broad-phase ([C63.2](02-broad-phase.md)).

## Queries: raycasts

The collision world ([C63.1](01-collision-world.md)) isn't only for *body-body* contact — it also answers
**queries**, chiefly **raycasts** ("what's along this ray?"):

- **Wheel ground tests** — each wheel raycasts *down* to find the ground ([C43.1](../C43-Collision-Contacts/01-detection.md))
  — the surface height and normal for the suspension ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md))
  and the surface tag ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)). This runs *every wheel, every
  frame* — a heavy use of raycasting.
- **AI line-of-sight** — the AI ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) raycasts to
  check visibility and avoidance ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)) — "is there a wall
  between me and my target?"
- **Placement/spawn tests** — checking whether a spawn point ([C61.3](../C61-Traffic-Ambient/03-spawning.md)) or a
  reset position ([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) is clear.

So the collision world is a *spatial query service* — bodies and AI ask it geometric questions (what's along this
ray, what's in this area) against the same structure ([C63.2](02-broad-phase.md)) that does body-body collision. A
raycast uses the grid ([C63.2](02-broad-phase.md)) to find the cells the ray passes through, then narrow-tests the
objects in them — the same broad/narrow pipeline ([C63.3](03-narrow-phase.md)) applied to a ray. This *shared
spatial structure* is efficient: one collision world, serving contacts *and* queries.

> 🟡 *Reasoned:* the raycast-over-the-grid and query-service role are the standard collision-world uses, consistent
> with the verified collision world and the wheel/AI systems ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md),
> [C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)); the exact query API is deeper RE. The collision
> world and cache are verified.

## The wheel raycast: the hot path

The heaviest query is the **wheel ground raycast** ([above](#queries-raycasts)) — every wheel, every frame, casts
down to find the road. This is the *hot path* of the collision world:

- **Four wheels × every car** — the player, cops, traffic ([Chapter 61](../C61-Traffic-Ambient/C61-Traffic-Ambient.md))
  — each wheel raycasts. With dozens of cars, that's *hundreds* of raycasts per frame.
- **Feeds the suspension and surface** — the raycast gives the ground contact
  ([C43.1](../C43-Collision-Contacts/01-detection.md)) that drives the suspension
  ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)) and the surface tag
  ([C44.1](../C44-Surfaces-Grip/01-surface-taxonomy.md)) — essential per-wheel data.
- **Must be fast** — because it runs so often, the wheel raycast must be *cheap* — leaning on the grid
  ([C63.2](02-broad-phase.md)) and cache ([above](#collisioncache-reuse-results)) to find the ground quickly.

So the wheel raycast is why the collision world's *performance* matters so much: it's queried hundreds of times per
frame just for the wheels ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)). The whole spatial
optimisation ([C63.2](02-broad-phase.md)) and caching ([above](#collisioncache-reuse-results)) exist partly to make
*this* — the per-wheel ground test — affordable at scale. The collision world isn't just for crashes; it's the
substrate every wheel stands on, queried constantly. This is the deepest reason collision performance
([C63.2](02-broad-phase.md)) is central: the cars are *always* touching the ground, and finding the ground is a
collision query.

## RE implications

- **`CollisionCache`** reuses collision results frame-to-frame (temporal coherence) — complementing the spatial
  broad-phase.
- **The collision world serves queries** — raycasts for wheel ground-tests, AI line-of-sight, spawn/reset checks.
- **The wheel ground raycast is the hot path** — every wheel, every frame (hundreds/frame) — feeding suspension and
  surface.
- **Shared spatial structure** — one collision world serves contacts and queries via the same grid/narrow pipeline.

---

### Key takeaways

- The **`CollisionCache`** reuses collision results **frame-to-frame** where geometry is unchanged — a **temporal**
  optimisation complementing the **spatial** broad-phase (only nearby + only changed = cheap in both dimensions).
- The collision world is a **spatial query service** — answering **raycasts** ("what's along this ray?") for wheel
  ground-tests, AI line-of-sight/avoidance, and spawn/reset checks — via the same grid/narrow pipeline.
- The **wheel ground raycast is the hot path** — every wheel, every car, every frame (**hundreds per frame**) —
  feeding the suspension and surface tag.
- The whole spatial optimisation and caching exist partly to make the **per-wheel ground test affordable at scale**
  — the cars are *always* on the ground, and finding it is a collision query.
- One **shared collision world** serves both body-body contacts and geometric queries — the spatial substrate of
  the whole physics.

**Continue:** [C63.5 — Reading the collision world in RE](05-reading-collision-world.md) · [Chapter 63 hub](C63-Collision-World.md)
