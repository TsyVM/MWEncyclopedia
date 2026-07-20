# Chapter 35 — Memory Management & Allocation

> **Goal of this chapter:** decode where objects come from in memory — the real zeroing pool allocator versus
> the getter-stub impostor that looks just like it, the `SlotPool` family and pre-sized pools, and the
> debug-fill sentinels that let you fingerprint globals in a memory dump.

Every live object ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) occupies memory, and
that memory comes from the engine's **allocator**. This chapter decodes the allocation layer — and it opens with
a cautionary tale of reverse-engineering: a two-line getter stub that a naive scan mistakes for the allocator.
Getting the allocator right, and knowing the pool structures, is what lets you reason about the game's memory.

> **Verified against the executable.** The **real object allocator is `0x5D29D0`** — confirmed from its bytes: a
> `__fastcall` (pool handle in `ECX`) that takes a size, compares it against `0x400`, and allocates+zeroes from
> a pool (`push ebx; mov ebx,[esp+8]; cmp ebx,0x400; …; mov edi,ecx`). The **impostor is `0x6269B0`** — verified
> to be exactly `mov eax, 0x009205E0; ret`, a getter stub that ignores its argument and returns a fixed global,
> mis-read as `operator new` because call sites push a size before it.

---

## Deep-dive pages

- [C35.1 — The real allocator vs the impostor](01-allocator-vs-impostor.md): `0x5D29D0` (real) and `0x6269B0`
  (getter stub).
- [C35.2 — Pool allocators & SlotPools](02-pools-slotpools.md): the `SlotPool` family and fixed-size pools.
- [C35.3 — Pre-sized pools](03-presized-pools.md): particle pools and the A* pools sized ahead of time.
- [C35.4 — Debug-fill sentinels](04-debug-fill.md): the fill bytes that fingerprint uninitialised/freed memory.
- [C35.5 — Allocation in the object lifecycle](05-lifecycle.md): allocate → construct → … → destruct → free.
- [C35.6 — Reading memory in RE](06-reading-memory.md): using pools and sentinels to map the heap.

---

## 35.1 The real allocator, and its impostor

The runtime has **one real object allocator and one impostor that looks just like it**
([C35.1](01-allocator-vs-impostor.md)):

- **`0x5D29D0` (real)** — a **zeroing pool allocator**: takes a pool handle (in `ECX`, `__fastcall`) and a size,
  allocates `size` bytes from the pool, **zeroes** them, and returns the block. The `cmp ebx, 0x400` is a
  size-class threshold (small vs large allocations routed differently).
- **`0x6269B0` (impostor)** — a two-line **getter stub** (`mov eax, 0x9205E0; ret`) that returns a fixed global
  and **ignores its pushed argument**. A naive scan keys on it because call sites push a size before calling —
  exactly as they would for `operator new` — but disassembly shows it does no allocation.

This is the chapter's cautionary lesson: **classify by behaviour, not by call-site shape**
([C34.2](../C34-VTable-Anatomy/02-classifying-slots.md)). The impostor is the memory-system version of a
mis-read vtable slot.

## 35.2 Pools, not a general heap

Objects come from **pools**, not a general-purpose heap ([C35.2](02-pools-slotpools.md)). The allocator
(`0x5D29D0`) allocates from a **pool handle**, and the engine has a family of **`SlotPool`** structures —
fixed-size-slot allocators that hand out and reclaim same-size blocks efficiently. Pooling suits a game: many
objects of the same size (particles, AI nodes, entities) are allocated and freed rapidly, and a slot pool does
that without fragmentation or general-heap overhead.

## 35.3 Some pools are pre-sized

Certain pools are **pre-allocated to a fixed capacity** at startup ([C35.3](03-presized-pools.md)) — notably the
**particle pools** and the **A\* pathfinding pools** ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)).
Pre-sizing bounds the memory a subsystem can use (no runtime growth, no fragmentation) and guarantees
allocation never fails mid-frame — the engine reserves the worst case up front. So a subsystem's pool size is a
hard cap on, e.g., how many particles or path nodes can exist at once.

## 35.4 Debug-fill sentinels fingerprint memory

The allocator and free paths write **debug-fill sentinels** — recognisable byte patterns into uninitialised or
freed memory ([C35.4](04-debug-fill.md)). These patterns (and the zeroing the real allocator does) let you
**fingerprint** regions in a memory dump: a block full of a fill byte is freed/uninitialised; a zeroed block is
freshly allocated; recognisable globals sit at known addresses ([C35.6](06-reading-memory.md)). So the fill
bytes are an RE gift — they mark the state of memory.

---

### Key takeaways

- The **real allocator is `0x5D29D0`** — a `__fastcall` zeroing pool allocator (pool handle in `ECX`, size
  vs `0x400` threshold).
- The **impostor `0x6269B0`** is a getter stub (`mov eax,0x9205E0; ret`) mis-read as `operator new` — classify
  by behaviour, not call-site shape.
- Objects come from **pools** (the `SlotPool` family), not a general heap — fixed-size slots, no fragmentation.
- Some pools are **pre-sized** at startup (particles, A\*) — bounding memory and guaranteeing allocation.
- **Debug-fill sentinels** (+ zeroing) fingerprint memory state (freed/fresh/global) in a dump.

**Next:** [Chapter 36 — Archives & the Virtual File System](../C36-Archives-VFS/C36-Archives-VFS.md): how bundles
become loadable resources.
