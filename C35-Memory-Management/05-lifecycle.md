# C35.5 — Allocation in the Object Lifecycle

> **The one-sentence version:** an object's life is allocate → construct → (configure → update each frame) →
> destruct → free — the allocator (`0x5D29D0`) and its pools bookend the lifecycle the registry, vault, and
> frame loop fill in.

[← C35.4 — Debug-fill sentinels](04-debug-fill.md) · [Chapter 35 hub](C35-Memory-Management.md) ·
[Next: C35.6 — Reading memory in RE →](06-reading-memory.md)

---

## The full lifecycle

Memory is one bookend of an object's life; the class system and frame loop fill in the rest:

```
allocate   (0x5D29D0, from a pool — C35.2)   → zeroed memory of `size` bytes
construct  (constructor — C33.3)             → vtable pointer set, fields initialised
configure  (vault data — C13, C12.5)         → the object's specific values applied
update     (each frame — Ch 37)              → the object's behaviour runs (C34.4)
…
destruct   (destructor — C33.3, C34.2)       → teardown, release owned resources
free       (return the slot to the pool)     → memory reclaimed, fill-marked (C35.4)
```

So the memory system ([C35.1](01-allocator-vs-impostor.md)–[C35.2](02-pools-slotpools.md)) provides the first
and last steps; the registry ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md))
provides construct/destruct; the vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) provides
configure; the frame loop ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) provides update.
The lifecycle is where all these chapters meet on one object.

## Allocate then construct

The first two steps are distinct and ordered ([C33.3](../C33-Class-Registry-Factories/03-construction.md)):

1. **Allocate** — the pool allocator ([C35.1](01-allocator-vs-impostor.md)) returns `size` **zeroed** bytes
   ([C35.4](04-debug-fill.md)) — raw, uninitialised memory.
2. **Construct** — the constructor writes the **vtable pointer** ([C34.1](../C34-VTable-Anatomy/01-what-is-a-vtable.md))
   and initialises the fields, turning raw memory into a valid object.

Between them, the block is a zeroed region with no vtable — a fingerprint of "allocated, not yet constructed"
([C35.4](04-debug-fill.md)). After construction, it's a live object identifiable by its vtable
([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)).

## Destruct then free

The end mirrors the beginning ([C33.3](../C33-Class-Registry-Factories/03-construction.md)):

1. **Destruct** — the destructor ([C34.2](../C34-VTable-Anatomy/02-classifying-slots.md)) runs teardown: release
   owned resources (other objects, handles), run field destructors.
2. **Free** — the block is returned to its pool ([C35.2](02-pools-slotpools.md)) and **fill-marked**
   ([C35.4](04-debug-fill.md)) — reclaimed for reuse.

Skipping the destructor (freeing without tearing down) leaks the object's owned resources; freeing without
returning to the pool leaks the slot. So the ordered destruct-then-free mirrors allocate-then-construct — memory
and behaviour bracketed symmetrically.

## Ownership and lifetime

An object's lifetime is governed by **ownership** — who holds it and decides when it dies:

- **A manager** ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) owns a
  population (cops, traffic) and constructs/destructs its members as the game demands.
- **Streaming** ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) owns
  section content and frees it when a section leaves residency.
- **Refcounting** ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) keeps a
  shared resource alive while any holder references it.

So an object isn't freed arbitrarily — its owner (a manager, streaming, or a refcount reaching zero) decides,
and then runs destruct→free. Tracing an object's lifetime means finding its owner.

> ✅ *Verified:* allocation is via the real pool allocator (`0x5D29D0`, [C35.1](01-allocator-vs-impostor.md)) with
> zeroing ([C35.4](04-debug-fill.md)); construct/destruct are the registry's constructor/destructor
> ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).
> 🟡 *Reasoned:* the ownership/refcount lifetime policies are the standard patterns, detailed for streaming in
> [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md); the allocate/construct/
> destruct/free bookends are verified.

## RE implications

- **The lifecycle spans chapters** — memory (allocate/free), registry (construct/destruct), vault (configure),
  frame (update).
- **Allocate-then-construct**: a zeroed no-vtable block becomes a live object ([C35.4](04-debug-fill.md)).
- **Destruct-then-free**: teardown then slot return; skipping either leaks.
- **Lifetime is ownership** — a manager, streaming, or refcount decides when an object dies; find the owner.

---

### Key takeaways

- Object life: **allocate → construct → configure → update → destruct → free**, bookended by the memory system.
- Allocate (zeroed pool memory) then construct (vtable + fields) — the two are distinct, ordered steps.
- Destruct (teardown owned resources) then free (return the fill-marked slot) — mirror of the start; skipping
  either leaks.
- **Lifetime is governed by ownership** — a manager, streaming, or a refcount decides when to destruct/free.
- The lifecycle is where memory, registry, vault, and frame-loop chapters meet on one object.

**Continue:** [C35.6 — Reading memory in RE](06-reading-memory.md) · [Chapter 35 hub](C35-Memory-Management.md)
