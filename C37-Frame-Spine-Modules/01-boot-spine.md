# C37.1 — The Boot Spine

> **The one-sentence version:** the game reaches its main loop through a short standard chain —
> `CRTStartup → WinMain → GameInit → FrameTick` — so the whole running game is reachable from the entry point by
> following four functions.

[← Chapter 37 hub](C37-Frame-Spine-Modules.md) · [Next: C37.2 — WinMain & the main loop →](02-winmain-loop.md)

---

## From the entry point

Disassembling `speed.exe` from its entry point forward recovers the boot chain — the sequence of functions that
gets from "the OS started the process" to "the game is running its frame loop":

```
0x7C4040  CRTStartup (__tmainCRTStartup)
             │  MSVC7 C runtime setup: GetVersionExA, GetCommandLineA, heap/CRT init
             ▼
0x666590  WinMain
             │  single-instance check (0x6CBC00, "speed.exe"); calls GameInit; then the main loop
             ▼
0x665FC0  GameInit
             │  one-time subsystem construction; opens the game package "NFS\ZDIR.BIN"/"NFS\ZZDATA"
             ▼
0x663D30  FrameTick   ← called every iteration of the main loop
```

Four functions are the entire structure of the running game. `CRTStartup` and `WinMain` are boilerplate (the C
runtime and the Windows entry); `GameInit` and `FrameTick` are the game — **build once, tick forever**.

## CRTStartup: the C runtime

`CRTStartup` (`0x7C4040`, the MSVC `__tmainCRTStartup`) is textbook C-runtime boilerplate: it queries the OS
(`GetVersionExA` `[0x8900E0]`), reads the command line (`GetCommandLineA` `[0x8901A4]`), sets up the heap and
CRT globals, and then calls the program's `WinMain` (`call 0x666590` at `0x7C41BE`), passing the result to
`exit`. It is not game code — it's the compiler's standard startup — but it's the outermost frame, so RE of the
boot chain starts here and immediately steps into `WinMain`.

## WinMain, GameInit, FrameTick: the game

The three game-relevant functions divide the running game cleanly:

- **`WinMain` (`0x666590`)** — the top-level: ensure a single instance, initialise the game (`GameInit`), then
  loop calling `FrameTick` until quit ([C37.2](02-winmain-loop.md)).
- **`GameInit` (`0x665FC0`)** — the **one-time** setup: construct every subsystem, open the data package
  ([C37.3](03-gameinit.md)).
- **`FrameTick` (`0x663D30`)** — the **per-frame** work: advance every subsystem one step
  ([C37.4](04-frametick.md)).

So the game is `init(); while(!quit) tick();` — the universal game-loop shape, and the boot spine is how you
find both halves in the binary.

> ✅ *Verified:* the boot chain `0x7C4040 → 0x666590 → 0x665FC0 → 0x663D30` is recovered by disassembling the
> entry point forward; `CRTStartup` calls `WinMain` at `0x7C41BE`.

## Why the spine is the RE entry point

The boot spine is where runtime reverse-engineering *starts*, because it's the one path guaranteed to reach
everything:

- **`GameInit` reaches every system's construction** — following its ~30 constructors
  ([C37.3](03-gameinit.md)) finds every subsystem's setup, and from a constructor you reach the class's vtable
  and behaviour ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **`FrameTick` reaches every system's update** — following its ~40 calls
  ([C37.5](05-module-order.md)) finds every subsystem's per-frame method.
- **Together they enumerate the engine** — every module is constructed in `GameInit` and ticked in `FrameTick`,
  so the two functions are a table of contents for the runtime.

So a subsystem you want to understand is somewhere in `GameInit`'s constructors (its setup) and `FrameTick`'s
calls (its update). The spine is the map ([C37.7](07-reading-spine.md)).

## RE implications

- **Start at the entry point** and follow to `WinMain` → `GameInit`/`FrameTick`.
- **`GameInit`'s constructors** enumerate subsystem setup; **`FrameTick`'s calls** enumerate subsystem updates.
- **The game is `init(); while(!quit) tick();`** — two functions are the whole structure.
- **The spine reaches everything** — it's the RE table of contents for the runtime.

---

### Key takeaways

- The boot chain is `CRTStartup (0x7C4040) → WinMain (0x666590) → GameInit (0x665FC0) → FrameTick (0x663D30)`.
- `CRTStartup` is MSVC C-runtime boilerplate; the game is `WinMain` + `GameInit` + `FrameTick`.
- Structure: **build once** (`GameInit`) then **tick forever** (`FrameTick`) — `init(); while(!quit) tick();`.
- `GameInit`'s ~30 constructors and `FrameTick`'s ~40 calls **enumerate the engine's modules**.
- The boot spine is the **entry point for runtime RE** — it reaches every subsystem.

**Continue:** [C37.2 — WinMain & the main loop](02-winmain-loop.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md)
