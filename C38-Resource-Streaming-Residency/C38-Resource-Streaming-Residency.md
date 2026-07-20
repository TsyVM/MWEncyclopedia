# Chapter 38 — The Resource Streaming Manager & Residency

> **Goal of this chapter:** decode how the game keeps the right data in memory at the right time — the StreamMgr
> singleton that owns the loaded-resource list, refcounted residency, the GameFlow phases that drive loading,
> and the preload manifests that make each phase ready.

The VFS ([Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)) finds a resource; the **streaming manager**
decides *when it's loaded* and *how long it stays*. It's the system that keeps the world resident around the
player ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)), preloads a phase's assets, and frees what's
no longer needed — the memory-over-time layer of the engine.

> **Verified against the executable.** The resource-streaming manager is the singleton at **`[0x91A098]`**
> (verified, 30 references), with its section list head at `[this+0x18]` and each section entry carrying a
> name/key pointer at `+0x08` (matched by `0x460D20`). Public forwarders wrap it: **`Stream_FindSection
> (0x507E40)`**, **`Stream_AcquireResources (0x5033C0)`**, **`Stream_ReleaseResources (0x503360)`** (37 callers
> — the ubiquitous "done with these assets"), and **`Stream_BlockUntilLoaded (0x503380)`** (loops
> `FindResidentSection`, pumping deferred callbacks while absent). ImageBase `0x400000`, RVA == file-offset.

---

## Deep-dive pages

- [C38.1 — The StreamMgr singleton](01-streammgr.md): the `[0x91A098]` manager and its section list.
- [C38.2 — Sections & residency](02-sections-residency.md): what's resident, and the section entry.
- [C38.3 — Refcounted acquire/release](03-refcounting.md): keeping a resource alive while anyone needs it.
- [C38.4 — GameFlow phases](04-gameflow.md): the phases that drive loading and unloading.
- [C38.5 — The preload manifests](05-manifests.md): the lists that make each phase ready.
- [C38.6 — Blocking loads & budgets](06-blocking-budgets.md): waiting for a resource, and load-time budgets.
- [C38.7 — Reading streaming in RE](07-reading-streaming.md): navigating the manager and residency.

---

## 38.1 The StreamMgr singleton

The streaming manager is a **singleton at `[0x91A098]`** — one global object, constructed in `GameInit`
([C37.3](../C37-Frame-Spine-Modules/03-gameinit.md)), that owns the set of **resident resources**. Its section
list head is at `[this+0x18]` — a linked list of loaded **sections** (units of resident data). All the streaming
forwarders ([C38.6](06-blocking-budgets.md)) operate on this one singleton, so `[0x91A098]` is the anchor for
everything about what's in memory ([C38.1](01-streammgr.md)).

## 38.2 Sections & residency

The unit of residency is a **section** — a loaded block of resources (a world section,
[Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md); a preload set; a car's assets). Each section entry
in the manager's list carries a **name/key pointer at `+0x08`** (matched by the comparison function `0x460D20`),
so the manager finds a resident section by key. "Is X loaded?" is a walk of the section list checking `+0x08`
against X's key ([C38.2](02-sections-residency.md)) — the `FindResidentSection` operation
([C38.6](06-blocking-budgets.md)).

## 38.3 Refcounted residency

Resources are kept resident by **reference counting** ([C38.3](03-refcounting.md)): `Stream_AcquireResources`
(`0x5033C0`) increments a resource's refcount (a system declares "I need these"), and `Stream_ReleaseResources`
(`0x503360`, **37 callers**) decrements it ("I'm done"). A resource stays resident while its refcount is
positive and is freed when it reaches zero. So residency is *demand-driven*: a section lives as long as any
system holds a reference. The 37 callers of `Release` show how pervasive this acquire/release discipline is.

## 38.4 GameFlow phases

Loading is driven by **GameFlow phases** ([C38.4](04-gameflow.md)) — the high-level states the game moves
through (boot, front-end, in-game, race, …). Each phase has a set of resources it needs; entering a phase
acquires them, leaving it releases them. So the *phase* is what decides *what's resident*: the front-end phase
holds the menu assets ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)), the in-game phase holds
the world ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)). Transitions are where big loads/unloads
happen.

## 38.5 The preload manifests

Each phase's resources are named by **preload manifests** ([C38.5](05-manifests.md)) — lists of what to load to
make a phase ready. These correspond to the `MemoryFile` manifests
([C36.4](../C36-Archives-VFS/04-memoryfile.md)) — Global, Permanent, InGame, and front-end — that scope
residency: the manifest names the files, and entering the phase acquires them (and makes the memory-resident
ones RAM-served). So a phase becomes ready by loading its manifest's contents, and the four preload manifests
are the four residency scopes.

## 38.6 Blocking loads and budgets

Most loading is **asynchronous** — a resource is requested and loaded over frames while the game continues. But
sometimes a resource is needed *now*: `Stream_BlockUntilLoaded` (`0x503380`) **waits** for a resident section,
looping `FindResidentSection` and **pumping deferred callbacks** (`RunDeferredCallbacks`) while it's absent — so
the load completes and the game proceeds ([C38.6](06-blocking-budgets.md)). Load-time **budgets** bound how much
streaming work runs per frame, keeping the frame rate stable during loads.

---

### Key takeaways

- The streaming manager is the **singleton `[0x91A098]`** (verified, 30 refs) — it owns the resident-section list
  (`[this+0x18]`).
- The unit of residency is a **section**, found by its name/key pointer at entry `+0x08` (compared by
  `0x460D20`).
- Residency is **refcounted**: `Stream_AcquireResources`/`Stream_ReleaseResources` (37 callers) keep a section
  alive while anyone needs it.
- **GameFlow phases** drive loading — each phase acquires its resources on entry and releases them on exit.
- **Preload manifests** (Global/Permanent/InGame/front-end) name each phase's resources; `Stream_BlockUntilLoaded`
  waits for a needed section, pumping callbacks.

**Next:** [Chapter 39 — Vehicle Simulation, End to End](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md): the
first of the simulation chapters, plugging into the frame.
