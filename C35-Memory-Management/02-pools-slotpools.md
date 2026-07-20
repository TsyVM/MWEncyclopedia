# C35.2 — Pool Allocators & SlotPools

> **The one-sentence version:** objects come from pools, not a general heap — the allocator takes a pool handle,
> and the `SlotPool` family hands out and reclaims fixed-size blocks, which suits a game's rapid same-size
> allocation without fragmentation.

[← C35.1 — The real allocator vs the impostor](01-allocator-vs-impostor.md) · [Chapter 35 hub](C35-Memory-Management.md) ·
[Next: C35.3 — Pre-sized pools →](03-presized-pools.md)

---

## Allocation is pool-based

The real allocator (`0x5D29D0`, [C35.1](01-allocator-vs-impostor.md)) doesn't allocate from a single global
heap — it allocates from a **pool**, identified by the **pool handle** passed in `ECX`. So the engine's memory
is organised as a set of **pools**, each serving a category of allocations, rather than one general heap. This is
a deliberate choice for a game, where allocation patterns are predictable and performance-critical.

## SlotPools: fixed-size slots

The core pool structure is the **`SlotPool`** family — allocators that manage **fixed-size slots**. A slot pool
holds an array of equal-size blocks and a free list; allocating takes a free slot, freeing returns it:

```
SlotPool (slot size S, capacity N)
├── slot 0  [used]
├── slot 1  [free] ─┐
├── slot 2  [used]  │ free list
├── slot 3  [free] ─┘
└── …
```

- **Allocate** — pop a slot from the free list (O(1)).
- **Free** — push the slot back (O(1)).
- **No fragmentation** — all slots are the same size, so freeing and re-allocating never fragments.

This is ideal for objects of one size allocated and freed rapidly — exactly a game's particles, AI nodes, and
entities.

## Why pools suit a game

A general heap (`malloc`/`free`) is flexible but fragments and is slow under churn. Pools trade flexibility for
speed and predictability, which a game wants:

- **Speed.** Slot allocation is O(1) pointer manipulation, far faster than a general heap's search.
- **No fragmentation.** Fixed-size slots mean freed memory is always reusable — critical over a long play
  session with millions of allocations.
- **Predictable memory.** A pool's capacity bounds its category's memory ([C35.3](03-presized-pools.md)) — you
  know the worst case.
- **Cache-friendliness.** Same-size objects packed in a pool are contiguous, improving cache behaviour when
  iterating them (e.g. updating all particles).

So the engine partitions memory into pools by category, each a `SlotPool` tuned to its objects' size and count.

> ✅ *Verified:* the real allocator (`0x5D29D0`) takes a pool handle in `ECX` ([C35.1](01-allocator-vs-impostor.md)),
> confirming pool-based allocation; the archive documents the `SlotPool` family.
> 🟡 *Reasoned:* the exact `SlotPool` struct layout (free-list head, slot size, capacity fields) is per-pool RE;
> the pool-handle allocation and fixed-size-slot model are verified/documented.

## Identifying a pool

Since the pool handle is the allocator's `ECX` argument ([C35.1](01-allocator-vs-impostor.md)), you identify
which pool an allocation uses by the handle at the call site:

- **The handle is a global** — a pool object at a fixed address; the same handle across call sites means the
  same pool.
- **The slot size** matches the objects it serves — a pool for 1964-byte cop cars
  ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) has 1964-byte slots.
- **The category** follows from what's allocated — a particle pool serves particles, an entity pool entities.

So tracing an allocation's pool handle tells you which category of memory it belongs to, and the pool's slot
size confirms the object's size ([C33.5](../C33-Class-Registry-Factories/05-fingerprints.md)).

## RE implications

- **Allocations are pool-based** — the `ECX` handle to `0x5D29D0` names the pool.
- **`SlotPool` = fixed-size slots + free list** — O(1) allocate/free, no fragmentation.
- **Identify a pool** by its handle (a global) and slot size ([C35.3](03-presized-pools.md)).
- **Pools partition memory by category** — particles, AI nodes, entities each have their pool.

---

### Key takeaways

- Objects come from **pools** (pool handle in `ECX`), not a general heap.
- The **`SlotPool`** family manages **fixed-size slots** with a free list — O(1) allocate/free, no
  fragmentation.
- Pools suit a game: fast, non-fragmenting, predictable, cache-friendly for same-size objects.
- Identify a pool by its handle (a global) and slot size, which matches its objects' size.
- Memory is partitioned into pools by category — particles, AI, entities each pooled.

**Continue:** [C35.3 — Pre-sized pools](03-presized-pools.md) · [Chapter 35 hub](C35-Memory-Management.md)
