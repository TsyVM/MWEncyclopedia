# C37.2 — WinMain & the Main Loop

> **The one-sentence version:** `WinMain` ensures a single instance, calls `GameInit`, then runs the main loop —
> `while (quit_flag [0x9257EC] == 0) { dt = scaled QPC delta; FrameTick(); }` — so the game is a real-time
> `while(!quit) tick()` over measured elapsed time.

[← C37.1 — The boot spine](01-boot-spine.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md) ·
[Next: C37.3 — GameInit & one-time construction →](03-gameinit.md)

---

## What WinMain does

`WinMain` (`0x666590`) is the game's top level, and it does exactly three things in order:

1. **Single-instance check** — it passes `"speed.exe"` to `0x6CBC00`, which ensures only one copy of the game
   runs (a named mutex / window search). If another instance exists, the game bails.
2. **`GameInit`** — the one-time construction of every subsystem ([C37.3](03-gameinit.md)).
3. **The main loop** — repeatedly time and tick the game until quit.

So `WinMain` is `check_single_instance(); GameInit(); main_loop();` — setup, then the loop that *is* the running
game.

## The main loop

The main loop, recovered from the disassembly, is the classic real-time game loop:

```
while (quit_flag [0x9257EC] == 0) {
    now  = QueryPerformanceCounter()
    dt   = scale(now - last)          // real elapsed seconds since last frame
    last = now
    FrameTick(0x663D30)               // advance the game by dt
}
```

Two globals govern it:

- **The quit flag `[0x9257EC]`** (10 references) — the loop condition. When any subsystem sets it (quit
  selected, window closed), the loop ends and the game shuts down.
- **The timestep** — computed each iteration from a **QueryPerformanceCounter** delta, scaled to seconds, and
  handed to `FrameTick` ([C37.4](04-frametick.md)) as `dt`.

So the loop runs as fast as it can, measuring real time each iteration, until told to quit.

> ✅ *Verified:* the quit flag `[0x9257EC]` has 10 references; `WinMain` calls `FrameTick` (`0x663D30`) in the
> loop; the single-instance check `0x6CBC00` receives `"speed.exe"`.

## Real elapsed time, not a fixed step

The loop times each frame from a QueryPerformanceCounter delta — **real elapsed time**, not a fixed step. This
matters for the simulation ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)):

- **Variable frame rate.** The loop runs each frame as fast as the machine allows; `dt` is however long the last
  frame took.
- **`FrameTick` scales by `dt`.** Because the timestep is passed in ([C37.4](04-frametick.md)), subsystems
  advance by real time — a slow frame advances the world further, a fast frame less, keeping motion consistent
  across frame rates.
- **Physics may sub-step.** Variable `dt` can destabilise integration, so the physics
  ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) may clamp or sub-divide it — but the loop
  itself is variable-rate.

So the game's clock is wall-clock time, sampled per frame, which is why the timestep is a shared global
([C37.4](04-frametick.md)) every time-dependent subsystem reads.

## The quit flag ends everything

The loop's exit is a single global, which makes shutdown clean:

- **Any subsystem can request quit** by setting `[0x9257EC]` — the menu ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)),
  the window handler ([C37.6](06-window-loop.md)) on close.
- **The loop checks it each iteration** — so quit takes effect at the next frame boundary, cleanly, not
  mid-frame.
- **After the loop**, `WinMain` tears down and returns to `CRTStartup`'s `exit` ([C37.1](01-boot-spine.md)).

So the game's lifetime is bounded by this one flag: constructed in `GameInit`, ticked in the loop, ended when the
flag is set.

## RE implications

- **`WinMain` = single-instance check + `GameInit` + main loop** — the top-level shape.
- **The main loop is `while(quit_flag==0) { dt=QPC delta; FrameTick(); }`** — real-time, variable-rate.
- **The quit flag `[0x9257EC]`** is the exit condition — any subsystem can set it; find its writers to find
  shutdown paths.
- **`dt` is a QPC delta** — the game runs on wall-clock time; subsystems scale by it
  ([C37.4](04-frametick.md)).

---

### Key takeaways

- `WinMain (0x666590)` does three things: single-instance check (`0x6CBC00`, "speed.exe"), `GameInit`, then the
  main loop.
- The main loop: **`while (quit_flag [0x9257EC] == 0) { dt = scaled QPC delta; FrameTick(); }`** — verified.
- The game runs on **real elapsed time** (QueryPerformanceCounter), variable-rate; subsystems scale by `dt`.
- The **quit flag `[0x9257EC]`** (10 refs) is the single exit condition — set it to end the game cleanly at a
  frame boundary.
- The loop is the running game; everything else is setup (`GameInit`) or per-frame work (`FrameTick`).

**Continue:** [C37.3 — GameInit & one-time construction](03-gameinit.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md)
