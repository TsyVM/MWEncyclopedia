# C35.3 — Pre-Sized Pools

> **The one-sentence version:** some pools are pre-allocated to a fixed capacity at startup — notably the
> particle pools and the A* pathfinding pools — so their memory is bounded, never fragments, and never fails to
> allocate mid-frame.

[← C35.2 — Pool allocators & SlotPools](02-pools-slotpools.md) · [Chapter 35 hub](C35-Memory-Management.md) ·
[Next: C35.4 — Debug-fill sentinels →](04-debug-fill.md)

---

## Reserved up front

Beyond ordinary pools ([C35.2](02-pools-slotpools.md)), certain pools are **pre-sized** — allocated to their
maximum capacity at startup and never grown. The engine reserves the worst case immediately, so the pool's
memory is fixed for the whole session. Two notable examples:

- **Particle pools** — the effects system ([C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md))
  reserves a fixed number of particle slots; effects draw from and return to it.
- **A\* pathfinding pools** — the road-network routing
  ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) reserves fixed pools for the A\* search's
  nodes/open-set, so pathfinding never allocates during a search.

## Why pre-size

Pre-sizing trades a fixed upfront reservation for three runtime guarantees a game values:

- **Bounded memory.** The pool can never exceed its reserved size, so a subsystem's memory footprint is a known
  constant — no runtime growth, no surprise spikes.
- **No fragmentation, ever.** A fixed pool of fixed slots ([C35.2](02-pools-slotpools.md)) never fragments and
  never needs to grow into fragmented space.
- **Allocation never fails mid-frame.** Because the worst case is already reserved, allocating a particle or a
  path node mid-frame is guaranteed to succeed (or the design guarantees it stays within capacity) — no
  mid-frame allocation stall or failure.

For real-time code, "allocation always succeeds and costs O(1)" is worth a fixed reservation — a stall or
failure during a frame is far worse than reserving memory that's sometimes unused.

## Capacity is a hard cap

The consequence of pre-sizing is that the pool's capacity is a **hard limit** on its category:

- The particle pool's size caps **how many particles can exist at once** — beyond it, new effects reuse/steal
  slots rather than allocate more.
- The A\* pool's size caps **how large a path search can be** — bounding the pathfinder's cost per query
  ([C18.5](../C18-Road-Network-CARP/05-routing.md)).

So these pool sizes are tuning knobs on the game's limits: raise the particle pool for denser effects (at more
memory), or the A\* pool for longer paths. The engine picks capacities that cover normal play; the caps are why
extreme situations degrade gracefully (particles recycled, paths bounded) rather than allocating unboundedly.

> ✅ *Verified (archive):* the engine uses pre-sized pools including particle pools and pre-sized A\* pools; the
> allocator is the verified pool allocator ([C35.1](01-allocator-vs-impostor.md)).
> 🟡 *Reasoned:* the exact reserved capacities and the recycle-when-full policy are per-pool detail; the
> pre-sized-pool design and the two examples are documented/verified.

## Pre-sized vs dynamic pools

The engine mixes both pool kinds:

- **Pre-sized pools** — fixed capacity, for subsystems where the worst case is known and mid-frame allocation
  must not fail (particles, A\*).
- **Dynamic pools** — grow as needed, for subsystems where the count varies more and occasional growth is
  acceptable (entities as the world streams, [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).

So a pool's *policy* (fixed vs growing) matches its category's needs — the real-time-critical, bounded ones are
pre-sized; the streaming, variable ones grow.

## RE implications

- **Pre-sized pools are reserved at startup** — find their allocation in `GameInit`
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)).
- **Capacity is a hard cap** — the pool size limits its category (particles, path nodes).
- **Sizes are tuning knobs** — raising a pre-sized pool raises its limit at a memory cost.
- **Distinguish pre-sized from dynamic** — real-time-critical pools are fixed; streaming ones grow.

---

### Key takeaways

- Some pools are **pre-sized** — reserved to max capacity at startup and never grown (particles, A\* pathfinding).
- Pre-sizing gives **bounded memory, no fragmentation, and guaranteed mid-frame allocation**.
- The capacity is a **hard cap** on the category (max particles, max path size) — degrading gracefully when
  full.
- Pool sizes are **tuning knobs** on the game's limits (more particles/longer paths cost memory).
- The engine mixes pre-sized (real-time-critical) and dynamic (streaming) pools by category.

**Continue:** [C35.4 — Debug-fill sentinels](04-debug-fill.md) · [Chapter 35 hub](C35-Memory-Management.md)
