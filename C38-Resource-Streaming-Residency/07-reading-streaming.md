# C38.7 — Reading Streaming in RE

> **The one-sentence version:** navigate streaming through the `[0x91A098]` singleton and its `Stream_*`
> forwarders — walk the section list to see what's resident, trace acquire/release to find holders, and follow
> phase transitions to the manifests — mapping the memory-over-time layer.

[← C38.6 — Blocking loads & budgets](06-blocking-budgets.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md) ·
[Next: Chapter 39 — Vehicle Simulation, End to End →](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)

---

## Anchors for streaming RE

Reverse-engineering the streaming system starts from fixed anchors:

- **The singleton `[0x91A098]`** ([C38.1](01-streammgr.md)) — the manager; its 30 references are the streaming
  code.
- **The section list at `[this+0x18]`** ([C38.2](02-sections-residency.md)) — the resident set; walk it to see
  what's loaded.
- **The forwarders** — `Stream_FindSection (0x507E40)`, `Stream_AcquireResources (0x5033C0)`,
  `Stream_ReleaseResources (0x503360)`, `Stream_BlockUntilLoaded (0x503380)` — the API
  ([C38.1](01-streammgr.md)).
- **The section key at `+0x08`** ([C38.2](02-sections-residency.md)) and its comparator `0x460D20`.

From these, the whole system is navigable: the manager, the resident list, the API, and the identity of each
section.

## The RE workflow

Reading streaming:

1. **Find the manager** at `[0x91A098]` and its list head at `[this+0x18]`
   ([C38.1](01-streammgr.md)).
2. **Enumerate resident sections** by walking the list, reading each `+0x08` key
   ([C38.2](02-sections-residency.md)).
3. **Trace acquire/release** — the 37 `Release` callers ([C38.3](03-refcounting.md)) are the resource holders;
   match acquires to releases.
4. **Follow phase transitions** ([C38.4](04-gameflow.md)) to the manifests ([C38.5](05-manifests.md)) — what
   each phase loads.
5. **Check the blocking path** — `Stream_BlockUntilLoaded` ([C38.6](06-blocking-budgets.md)) marks where the
   game waits (loading screens).

The output is the memory-over-time picture: what's resident, who holds it, and how it changes with phases.

## Streaming in a memory dump

Combined with memory RE ([C35.6](../C35-Memory-Management/06-reading-memory.md)), the streaming manager lets you
read residency from a **live dump**:

- **Walk `[0x91A098]+0x18`** to enumerate the resident sections — the actual loaded set at that moment.
- **Read each section's `+0x08` key** to identify it (a path-hash, [C36.3](../C36-Archives-VFS/03-binhash.md), or
  section id, [C15.2](../C15-Track-Streaming/02-section-table.md)).
- **Cross-reference the phase** ([C38.4](04-gameflow.md)) — the resident set should match the current phase's
  manifest ([C38.5](05-manifests.md)) plus streamed world sections.

So a dump's residency is legible: the manager's list *is* what's in memory, keyed for identification. This is
how you'd verify "did section X load?" or "what's resident during a race?" — read the list.

## Streaming ties the loading chapters together

The streaming manager is where the loading-side chapters converge:

- **The VFS** ([Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)) resolves *where* a resource is; streaming
  decides *when it's loaded and how long it stays*.
- **The world streaming** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) is the manager keeping
  world sections resident around the player.
- **The frame loop** ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) ticks the streamer
  each frame (one of `FrameTick`'s calls) within its budget ([C38.6](06-blocking-budgets.md)).
- **Memory** ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) is what streaming allocates and
  frees.

So streaming sits between the VFS (find) and memory (hold), driven by phases (when) and the frame loop (tick).
It's the coordinator of the loading substrate — the reason the right data is in memory at the right time.

## The substrate is complete

With streaming decoded, the **runtime substrate** (Part VII) is complete: the class system
([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)), registry
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)), vtables
([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)), memory
([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)), archives/VFS
([Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)), the frame spine
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)), and streaming (this chapter). Together
they are the machine the *content* runs on — objects constructed from loaded data, updated each frame, kept
resident by streaming. The next part ([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)+) reads
the content itself: the simulation and AI that run on this substrate.

## RE implications

- **Anchor on `[0x91A098]`, `[this+0x18]`, the `Stream_*` forwarders, and the `+0x08` key**.
- **Walk the section list** to enumerate residency (live, in a dump too).
- **Trace acquire/release** ([C38.3](03-refcounting.md)) for holders; **follow transitions** to manifests
  ([C38.5](05-manifests.md)).
- **Streaming coordinates the loading substrate** — VFS (find) + memory (hold) + phases (when) + frame (tick).

---

### Key takeaways

- Anchor streaming RE on `[0x91A098]`, the section list `[this+0x18]`, the `Stream_*` forwarders, and the
  `+0x08` key.
- **Walk the section list** to enumerate what's resident — legible in a live memory dump too.
- **Trace acquire/release** (37 `Release` callers) for holders; **follow phase transitions** to the manifests.
- Streaming is the **coordinator** between the VFS (find), memory (hold), phases (when), and the frame (tick).
- With streaming decoded, the **runtime substrate (Part VII) is complete** — the machine the simulation and AI
  run on.

**Continue:** [Chapter 39 — Vehicle Simulation, End to End](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md) ·
[Chapter 38 hub](C38-Resource-Streaming-Residency.md)
