# C37.3 — GameInit & One-Time Construction

> **The one-sentence version:** `GameInit` runs once at startup — it installs a callback, initialises the timer,
> opens the game package (`NFS\ZDIR.BIN`/`ZZDATA`), and runs ~30 back-to-back subsystem constructors — building
> every system the frame loop will tick.

[← C37.2 — WinMain & the main loop](02-winmain-loop.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md) ·
[Next: C37.4 — FrameTick & the timestep →](04-frametick.md)

---

## Build once

`GameInit` (`0x665FC0`) is the **one-time construction** phase — everything that must exist before the frame
loop can run. It executes once, between `WinMain`'s single-instance check and the main loop
([C37.2](02-winmain-loop.md)). Its verified structure:

```
GameInit (0x665FC0):
  install callback   [0x91DD84] = 0x64A650
  Timer::InitFrequency()                      // set up the QPC timing the loop uses
  open game package  "NFS\ZDIR.BIN" / "NFS\ZZDATA"   // the archive/VFS root (Ch 36)
  ~30 subsystem constructors                  // build every major system
```

So `GameInit` does the setup the loop depends on: timing (for `dt`, [C37.2](02-winmain-loop.md)), the data
package (for loading, [Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)), and the subsystems (for ticking,
[C37.5](05-module-order.md)).

## Opening the game package

A key `GameInit` step is **opening the game package** — `"NFS\ZDIR.BIN"` and `"NFS\ZZDATA"`. This is the
archive/VFS root ([Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)): `ZDIR.BIN` is the directory (the
path→resource index) and `ZZDATA` the packed data. Opening it makes the VFS operational, so every subsequent
load ([C36.6](../C36-Archives-VFS/06-loading.md)) — bundles, textures, vaults — can resolve. This must happen in
`GameInit` because the subsystem constructors that follow immediately load their data.

## The ~30 subsystem constructors

The bulk of `GameInit` is **~30 back-to-back constructors**, each building a major subsystem
([C37.5](05-module-order.md)):

- **Memory** ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — the allocator and pools, first,
  since everything else allocates.
- **Streaming** ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) — the
  StreamMgr singleton `[0x91A098]`.
- **Rendering, audio, physics, AI, input** — each system's manager/singleton constructed.
- **The class registry** ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) — the
  class registrations run here, populating the eleven family lists
  ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)).

Each constructor is a subsystem's setup; following them enumerates the engine's systems
([C37.1](01-boot-spine.md)). The order is deliberate — memory before things that allocate, streaming before
things that load — the construction analogue of the frame's update order ([C37.5](05-module-order.md)).

> ✅ *Verified:* `GameInit (0x665FC0)` installs `[0x91DD84]=0x64A650`, calls `Timer::InitFrequency`, opens
> `"NFS\ZDIR.BIN"`/`"NFS\ZZDATA"`, and runs ~30 constructors.
> 🟡 *Reasoned:* the specific subsystem each constructor builds is identified by following the constructor into
> its class ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)); the ~30-constructor structure and the
> package/timer/callback steps are verified.

## The callback and the timer

Two smaller `GameInit` steps set up the loop's machinery:

- **The callback `[0x91DD84] = 0x64A650`** — a function pointer the engine installs, likely a per-frame or event
  hook the loop or window handler ([C37.6](06-window-loop.md)) invokes. Installing it in `GameInit` wires it
  before the loop starts.
- **`Timer::InitFrequency`** — reads the QueryPerformanceCounter frequency so the main loop
  ([C37.2](02-winmain-loop.md)) can scale its QPC deltas to real seconds. Without it, `dt` would be raw counter
  ticks, not time.

So `GameInit` also prepares the timing and hooks the loop relies on — small but essential plumbing.

## Why one-time construction

Separating one-time setup (`GameInit`) from per-frame work (`FrameTick`) is fundamental engine structure:

- **Build the expensive things once.** Subsystems, singletons, and the VFS are constructed once, not per frame.
- **The loop stays lean.** `FrameTick` ([C37.4](04-frametick.md)) only *updates* the already-built systems — no
  construction in the hot path.
- **Clear lifetime.** Everything built in `GameInit` lives for the whole session; the loop ticks it; shutdown
  tears it down.

So `GameInit` and `FrameTick` are the two halves of the runtime: construct-once and update-per-frame. Reading
`GameInit` tells you *what exists*; reading `FrameTick` tells you *what runs each frame*.

## RE implications

- **`GameInit (0x665FC0)` is one-time construction** — callback, timer, game package, ~30 subsystem
  constructors.
- **It opens the VFS root** (`ZDIR.BIN`/`ZZDATA`) — required before any load
  ([Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)).
- **The ~30 constructors enumerate the subsystems** — follow each into its class
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **Construction order is deliberate** (memory → streaming → …) — mirroring the frame's update order
  ([C37.5](05-module-order.md)).

---

### Key takeaways

- `GameInit (0x665FC0)` is **one-time construction**: install callback `[0x91DD84]=0x64A650`, `Timer::InitFrequency`,
  open `NFS\ZDIR.BIN`/`ZZDATA`, ~30 subsystem constructors.
- **Opening the game package** makes the VFS operational (Chapter 36) before subsystems load their data.
- The **~30 constructors** build every major system and populate the class registry (Chapter 33) — following
  them enumerates the engine.
- Construction order is deliberate (memory first, streaming early) — mirroring the frame's dependency order.
- `GameInit` (build once) and `FrameTick` (update per frame) are the two halves of the runtime.

**Continue:** [C37.4 — FrameTick & the timestep](04-frametick.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md)
