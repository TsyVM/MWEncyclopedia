# C37.4 — FrameTick & the Timestep

> **The one-sentence version:** `FrameTick` is the per-frame heartbeat — it samples the timestep (`fild [esp+4]`)
> and scales it (`fmul`), stores it in the global `[0x9259BC]`, then makes 43 direct calls (41 unique) advancing
> every subsystem, before tail-jumping the frame-end forwarder.

[← C37.3 — GameInit & one-time construction](03-gameinit.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md) ·
[Next: C37.5 — The module update order →](05-module-order.md)

---

## The per-frame heartbeat

`FrameTick` (`0x663D30`) is called once per iteration of the main loop ([C37.2](02-winmain-loop.md)) and
advances the entire game by one frame. Its verified opening bytes tell the start of the story:

```
0x663D30  E8 0B 91 DF FF    call  0xREL        ; (a call)
          E8 16 39 00 00    call  0xREL        ; (a call)
          DB 44 24 04       fild  [esp+4]      ; load the timestep argument (dt, as int → float)
          51                push  ecx
          D8 …              fmul  …            ; scale the timestep
```

So `FrameTick` immediately **samples and scales the timestep** — the `dt` the main loop passed in
([C37.2](02-winmain-loop.md)) — `fild` loading it and `fmul` scaling it. Everything after runs the subsystems.

## The timestep global `[0x9259BC]`

The scaled timestep is stored in the global **`[0x9259BC]`** (23 engine-wide reads) — the shared `dt` every
time-dependent subsystem reads:

- **Physics** ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) integrates motion over `dt`.
- **Animations** ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md), [Chapter 26](../C26-World-Ambient-Animation/C26-World-Ambient-Animation.md))
  advance their clocks by `dt`.
- **Timers, effects, gameplay** advance by `dt`.

That 23 reads across the binary means `[0x9259BC]` is the game's canonical frame time — set once per frame by
`FrameTick`, read by every subsystem that moves with time. So the loop's QPC delta
([C37.2](02-winmain-loop.md)) flows into `FrameTick`, gets scaled, lands in `[0x9259BC]`, and drives the whole
simulation.

> ✅ *Verified:* `FrameTick (0x663D30)` opens with two `call`s then `fild [esp+4]` / `fmul` (sampling+scaling the
> timestep); the timestep global `[0x9259BC]` has 23 reads; `FrameTick` makes 43 direct calls (41 unique).

## The 43 calls: the subsystem ticks

After sampling `dt`, `FrameTick` makes **43 direct calls (41 unique)** — the "~40 subsystem ticks." Each call is
a subsystem's per-frame update: input, streaming, AI, vehicle simulation, physics, collision, rendering, audio,
HUD. So one frame is:

```
FrameTick(dt):
  sample + scale dt → [0x9259BC]
  call subsystem_1.tick()     ┐
  call subsystem_2.tick()     │ 43 calls (41 unique) — the module update order (C37.5)
  …                           │
  call subsystem_41.tick()    ┘
  tail-jump 0x6DF8E0          // frame-end forwarder (present, swap buffers, finalise)
```

The 43 calls (a few subsystems ticked twice, hence 41 unique) are the game's per-frame work, in a fixed
dependency order ([C37.5](05-module-order.md)). Following them enumerates every system's update method
([Chapter 34](../C34-VTable-Anatomy/04-method-roles.md)) — `FrameTick` is a table of contents for the running
game.

## The frame-end forwarder

`FrameTick` ends by **tail-jumping `0x6DF8E0`** — the frame-end forwarder. This is the end-of-frame work:
finalising rendering (present/swap the back buffer), advancing frame counters, running any deferred end-of-frame
callbacks. The tail-jump (rather than a call+return) is a small optimisation — `FrameTick`'s work is done, so it
hands the frame's finalisation to `0x6DF8E0` and lets that return to the main loop. So a frame is: sample dt →
tick subsystems → finalise (`0x6DF8E0`) → back to the loop.

## Why pass dt in

Passing the timestep into `FrameTick` (rather than each subsystem reading the clock) is deliberate
([C37.2](02-winmain-loop.md)):

- **One consistent dt per frame.** Every subsystem uses the *same* `[0x9259BC]` for the frame, so they advance
  in lockstep — physics and animation agree on how much time passed.
- **Frame-rate independence.** Because motion scales by the passed `dt`, the game runs consistently across frame
  rates ([C37.2](02-winmain-loop.md)) — a slow frame advances everything proportionally more.
- **Central control.** The loop can scale, clamp, or pause time by adjusting the `dt` it passes (e.g.
  Speedbreaker slow-motion, [plan Chapter 54](../README.md), sets a sim-rate), and every subsystem follows.

So `[0x9259BC]` is the single knob that controls the game's flow of time — set by `FrameTick`, read by all.

## RE implications

- **`FrameTick (0x663D30)` samples dt** (`fild [esp+4]`/`fmul`) into `[0x9259BC]`, then makes **43 calls (41
  unique)** — the subsystem ticks.
- **`[0x9259BC]` is the canonical frame time** (23 reads) — trace it to find every time-dependent subsystem.
- **The 43 calls enumerate per-frame updates** in dependency order ([C37.5](05-module-order.md)).
- **`FrameTick` tail-jumps `0x6DF8E0`** (frame-end: present/finalise) — the end of a frame.

---

### Key takeaways

- `FrameTick (0x663D30)` is the per-frame heartbeat: sample+scale the timestep (`fild`/`fmul`), then tick the
  subsystems.
- The scaled `dt` lives in **`[0x9259BC]`** (23 reads) — the canonical frame time every subsystem reads.
- It makes **43 direct calls (41 unique)** — the ~40 subsystem ticks in dependency order (C37.5).
- It ends by **tail-jumping `0x6DF8E0`** — the frame-end forwarder (present/finalise).
- Passing one `dt` in gives lockstep, frame-rate-independent time with a single control knob (`[0x9259BC]`).

**Continue:** [C37.5 — The module update order](05-module-order.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md)
