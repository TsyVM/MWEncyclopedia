# C38.4 — GameFlow Phases

> **The one-sentence version:** loading is driven by GameFlow phases — the high-level states the game moves
> through (boot, front-end, in-game, race) — each with a set of resources it acquires on entry and releases on
> exit, so the phase decides what's resident.

[← C38.3 — Refcounted acquire/release](03-refcounting.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md) ·
[Next: C38.5 — The preload manifests →](05-manifests.md)

---

## Phases drive loading

The game is a sequence of **GameFlow phases** — the coarse states it occupies:

```
boot → front-end (menus) → in-game (free-roam) → race → in-game → front-end → …
```

Each phase needs a **different set of resources**, and the streaming system loads/frees around phase
transitions. Entering a phase **acquires** ([C38.3](03-refcounting.md)) the resources it needs; leaving it
**releases** them. So the *phase* is the driver of residency: what's in memory at any moment is determined by
the current phase (plus per-frame streaming within it, [Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).

## Each phase's resource set

Phases map to resource sets:

- **Boot** — the minimal set to start (permanent data, [C38.5](05-manifests.md)).
- **Front-end** — the menu/shell assets ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)): UI
  atlases, fonts ([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)), the attract movie
  ([Chapter 23](../C23-Video-VP6/C23-Video-VP6.md)).
- **In-game** — the world ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)): the track, streamed
  sections, cars, gameplay data.
- **Race** — the event-specific additions on top of in-game.

So moving front-end → in-game releases the menu assets and acquires the world; the transition is a big
load/unload. This is why loading screens appear at phase transitions — the game is releasing one phase's
resources and acquiring the next's ([C38.6](06-blocking-budgets.md)).

> 🟡 *Reasoned:* the GameFlow phase model (phases with per-phase resource sets, acquired/released at
> transitions) is documented and consistent with the verified acquire/release forwarders
> ([C38.3](03-refcounting.md)) and the preload manifests ([C38.5](05-manifests.md)); the exact phase enumeration
> is per-system RE.

## Transitions are the load points

The **transitions** between phases are where the heavy streaming work happens:

- **Release the old phase** — decrement refcounts on the leaving phase's resources
  ([C38.3](03-refcounting.md)); sections whose count hits zero are freed.
- **Acquire the new phase** — increment refcounts on the entering phase's resources (from its manifest,
  [C38.5](05-manifests.md)); load what isn't resident.
- **Block until ready** — wait for the new phase's essential resources to load
  ([C38.6](06-blocking-budgets.md)) before the phase runs — the loading screen.

So a transition is release-then-acquire, bracketed by a wait. Shared resources (in both phases' sets) stay
resident across the transition (their refcount doesn't hit zero), so only the *difference* is actually
loaded/freed — an efficiency the refcounting provides ([C38.3](03-refcounting.md)).

## Residency scoped by phase

The upshot is that residency is **phase-scoped** — the same as the MemoryFile manifests
([C36.4](../C36-Archives-VFS/04-memoryfile.md)):

- **Permanent** — resident across all phases (never released).
- **Global** — resident broadly.
- **Front-end / In-game** — resident only in their phases.

So a resource's lifetime is tied to the phases that need it. Permanent data lives forever; phase data lives for
its phase. This scoping is what keeps memory bounded: only the current phase's (plus permanent) resources are
resident, not everything.

## RE implications

- **GameFlow phases drive loading** — each acquires its resources on entry, releases on exit
  ([C38.3](03-refcounting.md)).
- **Transitions are the load points** — release old + acquire new, bracketed by a wait (the loading screen).
- **Shared resources persist** across transitions (refcount stays positive) — only the difference loads/frees.
- **Residency is phase-scoped** — permanent/global/front-end/in-game, matching the manifests
  ([C38.5](05-manifests.md)).

---

### Key takeaways

- **GameFlow phases** (boot, front-end, in-game, race) drive loading — each has a resource set acquired on entry,
  released on exit.
- The **phase determines residency** — what's in memory is the current phase's resources (plus permanent) and
  per-frame streaming.
- **Transitions** are the heavy load points — release old + acquire new + block until ready (the loading
  screen).
- **Shared resources persist** across transitions via refcounting — only the difference is loaded/freed.
- Residency is **phase-scoped** (permanent/global/front-end/in-game), keeping memory bounded to the current
  phase.

**Continue:** [C38.5 — The preload manifests](05-manifests.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md)
