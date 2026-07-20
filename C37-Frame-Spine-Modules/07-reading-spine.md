# C37.7 — Reading the Frame Spine in RE

> **The one-sentence version:** navigate the runtime through the spine — enumerate subsystems from `GameInit`'s
> constructors and their updates from `FrameTick`'s calls, trace `dt` (`[0x9259BC]`) to find time-dependent
> systems, and use the spine as the table of contents for everything the game does.

[← C37.6 — The window & message loop](06-window-loop.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md) ·
[Next: Chapter 38 — The Resource Streaming Manager & Residency →](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)

---

## The spine is the map

The frame spine ([C37.1](01-boot-spine.md)) is the most useful navigation aid for runtime RE, because two
functions enumerate the whole engine:

- **`GameInit (0x665FC0)`** — its ~30 constructors ([C37.3](03-gameinit.md)) are every subsystem's **setup**.
- **`FrameTick (0x663D30)`** — its 43 calls ([C37.4](04-frametick.md)) are every subsystem's **update**.

So any subsystem X has a construction in `GameInit` and an update in `FrameTick`. To understand X: find its
constructor (its setup and class, [Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) and its tick (its
per-frame behaviour). The spine turns "where is system X?" into "which `GameInit` constructor and which
`FrameTick` call?"

## The RE workflow

Reading the runtime through the spine:

1. **From the entry point** ([C37.1](01-boot-spine.md)) reach `WinMain` → `GameInit`/`FrameTick`.
2. **Enumerate subsystems** from `GameInit`'s constructors ([C37.3](03-gameinit.md)) — follow each into its class
   ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).
3. **Enumerate updates** from `FrameTick`'s 43 calls ([C37.4](04-frametick.md),
   [C37.5](05-module-order.md)) — each is a subsystem's tick.
4. **Match construction to update** — a subsystem's constructor and its tick are the same system's two spine
   appearances.
5. **Trace `dt`** (`[0x9259BC]`, [C37.4](04-frametick.md)) — its 23 readers are the time-dependent subsystems.

The output is the engine mapped as a set of subsystems, each with a setup, an update, and a place in the frame
order.

## Anchors

The spine offers fixed anchors for RE:

- **The boot chain** — `0x7C4040` → `0x666590` → `0x665FC0` → `0x663D30` ([C37.1](01-boot-spine.md)).
- **The quit flag `[0x9257EC]`** — the loop condition ([C37.2](02-winmain-loop.md)); its writers are shutdown
  paths.
- **The timestep `[0x9259BC]`** — the frame time ([C37.4](04-frametick.md)); its readers are time-dependent
  systems.
- **`WndProc (0x6DB6C0)`** — the OS boundary ([C37.6](06-window-loop.md)).
- **The frame-end forwarder `0x6DF8E0`** — end-of-frame ([C37.4](04-frametick.md)).

These are stable entry points; from them the whole runtime is reachable.

## The spine ties the runtime together

The frame spine is where the runtime-substrate chapters converge:

- **Memory** ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — constructed first in `GameInit`.
- **The class registry** ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) —
  populated in `GameInit`.
- **Streaming** ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) —
  constructed in `GameInit`, ticked in `FrameTick`.
- **The VFS** ([Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)) — opened in `GameInit`.

So `GameInit` and `FrameTick` are the frame the substrate chapters plug into: build memory, the registry, the
VFS, and streaming, then tick them each frame. The spine is the skeleton; the substrates are the organs.

## The spine leads to the simulation

`FrameTick`'s calls are the entry points to the *content* chapters that follow:

- The **vehicle sim** ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) is one of `FrameTick`'s
  calls — its per-frame pipeline runs here.
- The **physics** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) integrates over `dt`
  (`[0x9259BC]`) in another call.
- The **AI** ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), **collision**
  ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)), and the rest each occupy a call.

So the simulation and AI chapters ([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)+) are, from
the spine's vantage, guided reads of specific `FrameTick` calls — each chapter following one call into its
subsystem. The spine is where they all begin: the frame that runs them.

## RE implications

- **The spine enumerates the engine** — `GameInit` constructors (setup) + `FrameTick` calls (update).
- **Match construction to update** — a subsystem's two spine appearances are its setup and its tick.
- **Trace `dt` (`[0x9259BC]`) and the quit flag (`[0x9257EC]`)** — to find time-dependent systems and shutdown
  paths.
- **`FrameTick`'s calls are the entry points** to the simulation/AI chapters
  ([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)+).

---

### Key takeaways

- The spine is the runtime **map**: `GameInit`'s constructors (subsystem setup) + `FrameTick`'s calls (subsystem
  updates).
- Understand a subsystem by matching its **`GameInit` constructor** to its **`FrameTick` tick**.
- Anchors: the boot chain, the quit flag `[0x9257EC]`, the timestep `[0x9259BC]`, `WndProc`, the frame-end
  forwarder.
- The spine is where the **substrate chapters** (memory, registry, VFS, streaming) plug in — build then tick.
- `FrameTick`'s calls are the **entry points** to the simulation/AI chapters that follow.

**Continue:** [Chapter 38 — The Resource Streaming Manager & Residency](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md) ·
[Chapter 37 hub](C37-Frame-Spine-Modules.md)
