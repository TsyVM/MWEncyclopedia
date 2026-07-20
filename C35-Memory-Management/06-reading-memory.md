# C35.6 — Reading Memory in RE

> **The one-sentence version:** map the heap by anchoring on the real allocator and pools, fingerprinting
> regions by their byte patterns (zeroed/filled/live/global), and identifying live objects by their vtable
> pointers — turning a memory dump into a labelled structure.

[← C35.5 — Allocation in the object lifecycle](05-lifecycle.md) · [Chapter 35 hub](C35-Memory-Management.md) ·
[Next: Chapter 36 — Archives & the Virtual File System →](../C36-Archives-VFS/C36-Archives-VFS.md)

---

## Anchors for memory RE

Reading the game's memory starts from fixed anchors:

- **The real allocator `0x5D29D0`** ([C35.1](01-allocator-vs-impostor.md)) — trace its callers to find where
  each category of object is allocated, and its `ECX` pool handle to know which pool.
- **The pools** ([C35.2](02-pools-slotpools.md)) — global pool objects; their slot sizes match their objects.
- **Globals** — the family list-heads ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)), singletons
  (StreamMgr at `0x91A098`, [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)),
  and fixed addresses like the impostor's `0x9205E0`.
- **VTables** — fixed-address tables that identify live objects
  ([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)).

From these, the heap becomes navigable — you know where objects come from, where they live, and how to name
them.

## Fingerprint, then identify

The two-step method for a memory dump ([C35.4](04-debug-fill.md)):

1. **Fingerprint regions** by byte pattern — zeroed (fresh), fill-byte (dead), varied-with-vtable (live),
   stable-at-known-address (global). This partitions the dump into interesting (live/global) and skippable
   (dead/fresh).
2. **Identify live objects** — for each region starting with a vtable pointer, look up the vtable in the
   catalogue ([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)) to name the class, confirm with size
   ([C33.5](../C33-Class-Registry-Factories/05-fingerprints.md)), and read its fields.

So a raw dump becomes a labelled map: "this is an `AIVehicleCopCar` at address X, these are its fields, this
region is freed." The fingerprinting focuses effort; the vtable identification names the targets.

## Following references

Objects reference other objects (via field pointers), so once you've identified one, you follow its **field
pointers** to reach others:

```
identified object (AIVehicleCopCar)
├── field: pointer → its mechanics (Ch 40)
├── field: pointer → its AI goal (Ch 46)
├── field: pointer → its rigid body (Ch 41)
└── …
```

Each pointer, resolved and identified ([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)), extends the map
to a connected object. So from one anchor you traverse the object graph — the cop car to its mechanics to its
physics to its AI — reconstructing the runtime's live structure. This is how the simulation and AI chapters
([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
map the relationships between classes.

## Cross-referencing static and live

Memory RE (live) complements binary RE (static):

- **Static** ([Chapters 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)–[34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md))
  — the classes, vtables, and code in `speed.exe`.
- **Live** (this page) — the actual objects and their field values at runtime.

Static tells you *what a class is and does*; live tells you *what a specific instance's state is*. Together they
give the full picture: a cop car's class (static) and this cop car's speed/position/goal (live). The vtable is
the join — it identifies a live object as a static class ([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)).

## The memory-RE checklist

- **Anchor** on the allocator, pools, globals, and vtables.
- **Fingerprint** the dump by byte pattern (fresh/dead/live/global).
- **Identify** live objects by vtable pointer + size.
- **Follow** field pointers to traverse the object graph.
- **Cross-reference** static (class/behaviour) and live (instance state).

This turns the running game's memory into a labelled, navigable structure — the endpoint of the runtime-substrate
chapters and the starting point for the simulation and AI chapters that read specific systems' live state.

---

### Key takeaways

- Anchor memory RE on the **real allocator, pools, globals, and vtables** — fixed points to navigate from.
- **Fingerprint then identify**: partition the dump by byte pattern, then name live objects by vtable pointer +
  size.
- **Follow field pointers** to traverse the object graph (cop car → mechanics → physics → AI).
- **Cross-reference static and live** — the class/behaviour (binary) and the instance state (memory), joined by
  the vtable.
- The method turns a raw dump into a labelled structure — the endpoint of the substrate chapters.

**Continue:** [Chapter 36 — Archives & the Virtual File System](../C36-Archives-VFS/C36-Archives-VFS.md) ·
[Chapter 35 hub](C35-Memory-Management.md)
