# Chapter 37 — The Frame Spine & Engine Modules

> **Goal of this chapter:** trace the running game's backbone — the boot chain that reaches the main loop, the
> per-frame `FrameTick` that advances every subsystem, and the module update order — so you know exactly where
> each system plugs into the frame.

The class system ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) makes objects; the
**frame spine** makes them *live*. Every frame the engine samples a timestep and calls each subsystem in a
fixed order — the heartbeat that turns a pile of objects into a running game. This chapter is that spine, read
from the executable's entry point forward.

> **Verified against the executable.** The boot chain is `CRTStartup (0x7C4040) → WinMain (0x666590) → GameInit
> (0x665FC0) → FrameTick (0x663D30)`. Confirmed live: `FrameTick` at `0x663D30` opens with a `call`
> (`E8 0B 91 DF FF`), a second `call`, then `DB 44 24 04` (`fild [esp+4]` — sampling the timestep) and `D8`
> (`fmul`); it makes **43 direct calls (41 unique)** — the "~40 subsystem ticks." The main-loop **quit flag
> `[0x9257EC]`** has 10 references, and the **timestep global `[0x9259BC]`** has 23 engine-wide reads. ImageBase
> `0x400000`, RVA == file-offset.

---

## Deep-dive pages

- [C37.1 — The boot spine](01-boot-spine.md): `CRTStartup → WinMain → GameInit → FrameTick`, from the entry
  point.
- [C37.2 — WinMain & the main loop](02-winmain-loop.md): the single-instance check, the quit flag, and the QPC
  timing.
- [C37.3 — GameInit & one-time construction](03-gameinit.md): the ~30 startup constructors and the game package.
- [C37.4 — FrameTick & the timestep](04-frametick.md): sampling `dt`, scaling it, and the 43 per-frame calls.
- [C37.5 — The module update order](05-module-order.md): the ~40 subsystem ticks and why order matters.
- [C37.6 — The window & message loop](06-window-loop.md): `WndProc`, focus, and the OS boundary.
- [C37.7 — Reading the frame spine in RE](07-reading-spine.md): navigating the boot chain and tick in
  `speed.exe`.

---

## 37.1 The boot spine

The game reaches its main loop through a short, standard chain, recovered by disassembling the entry point
forward:

```
0x7C4040  CRTStartup   (__tmainCRTStartup)   MSVC7 CRT setup, then calls WinMain
0x666590  WinMain      single-instance check (0x6CBC00, "speed.exe"), GameInit, then the main loop
0x665FC0  GameInit     one-time subsystem construction (~30 constructors), opens the game package
0x663D30  FrameTick    the per-frame tick — samples dt, runs ~40 subsystem updates
```

Everything the game does is reachable from this spine ([C37.1](01-boot-spine.md)): the one-time setup in
`GameInit`, and the per-frame work in `FrameTick`. Two functions — `GameInit` and `FrameTick` — are the whole
structure: *build once*, then *tick forever*.

## 37.2 The main loop

`WinMain` (`0x666590`) does three things: a **single-instance check** (`0x6CBC00`, passing `"speed.exe"`) so two
copies don't run, a call to `GameInit`, and then the **main loop** ([C37.2](02-winmain-loop.md)):

```
while (quit_flag [0x9257EC] == 0) {
    dt = scaled QueryPerformanceCounter delta      // real elapsed time
    FrameTick(0x663D30)                            // advance the game one frame
}
```

The loop runs until the **quit flag `[0x9257EC]`** (10 references) is set, timing each frame from a
QueryPerformanceCounter delta. So the game is a `while(!quit) tick()` loop over real elapsed time — the classic
game-loop shape, here in the binary ([C37.2](02-winmain-loop.md)).

## 37.3 GameInit: build once

`GameInit` (`0x665FC0`) is the **one-time construction** phase ([C37.3](03-gameinit.md)): it installs a callback
(`[0x91DD84] = 0x64A650`), initialises the timer (`Timer::InitFrequency`), **opens the game package**
(`"NFS\ZDIR.BIN"` / `"NFS\ZZDATA"` — the archive/VFS root,
[Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)), and runs **~30 back-to-back subsystem constructors** —
each building a major system (rendering, audio, streaming, physics, AI). This is where the class registry
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) is populated and the singletons
(like StreamMgr `[0x91A098]`, [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md))
are created. Build once, at startup.

## 37.4 FrameTick: tick forever

`FrameTick` (`0x663D30`) is the **per-frame heartbeat** ([C37.4](04-frametick.md)). Its verified opening samples
the timestep (`fild [esp+4]`) and scales it (`fmul`), then it makes **43 direct calls (41 unique)** — the ~40
subsystem ticks — before tail-jumping the frame-end forwarder `0x6DF8E0`. So one frame is: **sample dt → update
every subsystem in order → finish**. The timestep global `[0x9259BC]` (23 reads) is `dt` shared across the
subsystems that need it — physics ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) integrates
over it, animations advance by it.

## 37.5 The module update order

The ~40 calls in `FrameTick` run the subsystems in a **fixed order** ([C37.5](05-module-order.md)), and the
order matters: input before simulation before rendering, physics before collision response, AI before the
vehicles it drives. So `FrameTick` is not an arbitrary list — it's the **dependency order** of the engine's
modules, each plugged in at the point its inputs are ready. Reading that order is reading how the game's systems
depend on one another.

## 37.6 The window loop

Alongside the game loop, `WndProc` (`0x6DB6C0`) handles the **OS window messages**
([C37.6](06-window-loop.md)) — `WM_PAINT`, `WM_ACTIVATE` (which emits a `"LostFocus"` event so the game can
pause/mute when backgrounded), and the rest, via a jump table. This is the boundary between the OS and the game:
the message loop feeds window events in, the game loop ticks the simulation.

---

### Key takeaways

- The boot spine is `CRTStartup (0x7C4040) → WinMain (0x666590) → GameInit (0x665FC0) → FrameTick (0x663D30)` —
  verified.
- The structure is **build once** (`GameInit`, ~30 constructors, opens the game package) then **tick forever**
  (`FrameTick`).
- The **main loop** runs `while(quit_flag [0x9257EC]==0)`, timing frames from a QueryPerformanceCounter delta.
- `FrameTick` samples the timestep (`fild`/`fmul`) and makes **43 calls (41 unique)** — the ~40 subsystem ticks
  in dependency order; `dt` is the global `[0x9259BC]` (23 reads).
- `WndProc (0x6DB6C0)` handles OS messages (`WM_ACTIVATE` → `"LostFocus"`) — the window/game boundary.

**Next:** [Chapter 38 — The Resource Streaming Manager & Residency](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md):
how resources become and stay resident across the frame.
