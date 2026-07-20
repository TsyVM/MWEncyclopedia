# C37.5 — The Module Update Order

> **The one-sentence version:** the ~40 calls in `FrameTick` run the subsystems in a fixed dependency order —
> input before simulation before rendering, AI before the vehicles it drives, physics before collision response
> — so the order *is* the engine's module dependency graph.

[← C37.4 — FrameTick & the timestep](04-frametick.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md) ·
[Next: C37.6 — The window & message loop →](06-window-loop.md)

---

## The order is not arbitrary

`FrameTick`'s 43 calls ([C37.4](04-frametick.md)) aren't a random list — they run in a **fixed dependency
order**, because each subsystem needs the outputs of the ones before it. A plausible frame order, following the
dependencies:

```
FrameTick:
  1. input        — read the controller/keyboard (Ch 53 plan)
  2. streaming    — service resource loads (Ch 38)
  3. AI           — decide what non-player entities do (Ch 46) — needs input/world state
  4. vehicle sim  — advance every vehicle (Ch 39) — needs AI + input
  5. physics      — integrate motion (Ch 41) — needs the forces the sim produced
  6. collision    — detect + respond (Ch 43) — needs the new positions
  7. camera       — frame the scene (Ch 52 plan) — needs the final positions
  8. rendering    — draw the world (Ch 61 plan) — needs everything placed
  9. audio        — mix the frame's sound (Ch 19) — needs the frame's events
 10. HUD          — draw the overlay (Ch 27) — needs the final state
     … (43 calls total, some subsystems ticked more than once)
```

The rule is **producers before consumers**: a subsystem runs after the ones whose output it reads. So the order
encodes the engine's module dependency graph — read it and you know which systems feed which.

## Why order matters

Getting the order right is what makes a frame *correct*:

- **Input before simulation.** The vehicle sim ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
  must use *this frame's* input, so input reads first.
- **AI before vehicles.** A cop's AI ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) decides its
  throttle/steer, then the vehicle sim applies it — AI produces, the vehicle consumes.
- **Physics before collision.** Motion integrates ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)),
  *then* collisions are detected against the new positions and resolved
  ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)).
- **Everything before rendering.** The renderer draws the *final* state, so it runs last (before HUD/present).

Run these out of order — render before simulate, collide before move — and the frame shows stale state or
resolves collisions against old positions. The fixed order is the engine guaranteeing each subsystem sees
up-to-date inputs.

> 🟡 *Reasoned:* the specific ordering (input → AI → sim → physics → collision → render → audio → HUD) is the
> standard game-frame dependency order, consistent with the verified 43-call `FrameTick`
> ([C37.4](04-frametick.md)) and the subsystems it ticks; the exact per-call assignment is recovered by
> following each call into its subsystem ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)). The
> 43-call structure and `dt`-first sampling are verified.

## Where a subsystem plugs in

For any subsystem, its **place in `FrameTick`** tells you when it runs relative to the others — which is often
the key to understanding it:

- A subsystem's **inputs** are whatever ran *before* it in `FrameTick`.
- Its **outputs** are consumed by whatever runs *after*.
- Its **timing** is the shared `dt` ([C37.4](04-frametick.md)) — it advances by the frame's time.

So "where does system X plug into the frame?" is answered by finding its call in `FrameTick`'s 43. The vehicle
sim ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) plugs in after AI and input, before
physics finalisation — which is exactly the pipeline that chapter describes, seen from the frame's vantage.

## Some subsystems tick more than once

The 43 calls but 41 unique ([C37.4](04-frametick.md)) means a couple of subsystems are ticked **more than once
per frame** — a common pattern where a system needs a pre-pass and a post-pass:

- **A pre-update and a post-update** — e.g. a subsystem that reads state early and applies results late.
- **A fixed sub-step** — physics ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) may run
  multiple sub-steps per frame for stability, appearing as repeated calls.

So the small gap between 43 and 41 is informative: it flags the subsystems that need multiple passes within a
frame, usually for correctness (physics stability) or ordering (pre/post split).

## RE implications

- **`FrameTick`'s call order is the module dependency graph** — producers before consumers.
- **Find a subsystem's call** to learn when it runs, its inputs (earlier calls), and its consumers (later
  calls).
- **The order guarantees fresh inputs** — input→AI→sim→physics→collision→render; out-of-order breaks the frame.
- **43 vs 41** flags subsystems ticked twice (pre/post pass or physics sub-steps).

---

### Key takeaways

- `FrameTick`'s ~40 calls run in a **fixed dependency order** — the engine's module dependency graph.
- The rule is **producers before consumers**: input → AI → vehicle sim → physics → collision → render → audio →
  HUD.
- Order guarantees each subsystem sees **fresh inputs**; wrong order shows stale state or mis-resolved
  collisions.
- A subsystem's **place in `FrameTick`** tells you its inputs (earlier), consumers (later), and timing (`dt`).
- **43 calls but 41 unique** flags subsystems ticked twice (pre/post pass or physics sub-steps).

**Continue:** [C37.6 — The window & message loop](06-window-loop.md) · [Chapter 37 hub](C37-Frame-Spine-Modules.md)
