# C38.3 — Refcounted Acquire/Release

> **The one-sentence version:** residency is reference-counted — `Stream_AcquireResources` says "I need these"
> (refcount++), `Stream_ReleaseResources` says "I'm done" (refcount--, 37 callers), and a section stays resident
> while its count is positive, freed at zero.

[← C38.2 — Sections & residency](02-sections-residency.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md) ·
[Next: C38.4 — GameFlow phases →](04-gameflow.md)

---

## Demand-driven residency

A section stays resident because **something needs it**, tracked by a **reference count**:

- **`Stream_AcquireResources` (`0x5033C0`)** — a system declares "I need these resources": increment the
  refcount (and load if not resident).
- **`Stream_ReleaseResources` (`0x503360`)** — a system declares "I'm done with these": decrement the refcount.
- **Free at zero** — when a section's refcount drops to zero (no system needs it), it's freed and its memory
  reclaimed ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)).

So residency is **demand-driven**: a section lives exactly as long as at least one system holds a reference to
it. This is the standard reference-counting lifetime ([C35.5](../C35-Memory-Management/05-lifecycle.md)) applied
to loaded resources.

## Acquire/release is pervasive

`Stream_ReleaseResources` has **37 callers** — verified — which shows how ubiquitous the acquire/release
discipline is. Every system that uses streamed resources brackets its use:

```
Stream_AcquireResources(resources)     // I need these — load / refcount++
   … use the resources (draw, simulate) …
Stream_ReleaseResources(resources)     // I'm done — refcount--, free if zero
```

The 37 `Release` callers are the systems that hold resources: the world streamer
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)), the front-end
([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)), car loaders, effect systems. Each acquires
what it needs and releases when done, and the manager frees a section only when *all* its holders have released.
So "done with these assets" is one of the most common streaming operations in the game.

> ✅ *Verified:* `Stream_AcquireResources (0x5033C0)` and `Stream_ReleaseResources (0x503360)` are the
> acquire/release forwarders; `Release` has **37 callers** — the acquire/release discipline is pervasive.

## Why refcount instead of ownership

Reference counting (many holders) rather than single ownership suits **shared** resources:

- **Sharing.** A texture or model needed by several systems (e.g. a shared car template,
  [C5.3](../C5-Textures-TPK/03-two-variants.md)) is held by each; it stays resident while *any* holds it, freed
  only when *all* release.
- **Overlap.** As the player moves, a world section may be needed by the streamer and by an entity in it
  simultaneously; refcounting keeps it resident until neither needs it.
- **Safety.** No holder frees a resource another is still using — the count prevents premature free (a
  use-after-free, [C35.4](../C35-Memory-Management/04-debug-fill.md)).

So refcounting is the right lifetime for shared, overlapping resource needs — the common case in a streaming
open world. Single ownership would force one system to decide when a shared resource dies, which it can't
safely.

## The acquire/release contract

The correctness of streaming rests on a **balanced acquire/release contract**:

- **Every acquire must be matched by a release.** Acquire without release **leaks** — the section never reaches
  zero refcount and stays resident forever, wasting memory.
- **Release without acquire** under-counts — the section may be freed while still in use (a crash /
  use-after-free).
- **Phase transitions** ([C38.4](04-gameflow.md)) are where large acquire/release batches happen — entering a
  phase acquires its manifest ([C38.5](05-manifests.md)), leaving it releases.

So the acquire/release pairs must balance, exactly like new/delete or malloc/free — an unbalanced pair is a leak
or a crash. The 37 `Release` callers are the counterparts to the acquires scattered through the systems.

## RE implications

- **Residency is refcounted** — `Acquire` (`0x5033C0`, ++) and `Release` (`0x503360`, --, 37 callers); free at
  zero.
- **Acquire/release is pervasive** — every resource user brackets its use; the 37 `Release` callers are the
  holders.
- **Refcounting suits shared resources** — a section stays resident while any holder needs it, safely.
- **The contract must balance** — unmatched acquire leaks, unmatched release crashes; phase transitions batch
  them.

---

### Key takeaways

- Residency is **reference-counted**: `Stream_AcquireResources` (refcount++) and `Stream_ReleaseResources`
  (refcount--); free at zero.
- `Release` has **37 callers** — acquire/release is pervasive; every resource user brackets its use.
- Refcounting suits **shared, overlapping** resources — a section stays resident while any holder needs it,
  safely.
- The **acquire/release contract must balance** — unmatched acquire leaks memory, unmatched release causes a
  use-after-free.
- Phase transitions ([C38.4](04-gameflow.md)) batch acquire/release for a phase's manifest
  ([C38.5](05-manifests.md)).

**Continue:** [C38.4 — GameFlow phases](04-gameflow.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md)
