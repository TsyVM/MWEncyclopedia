# Chapter 64 — World Update: Bodies, Effects & the Active Lists

> **Goal of this chapter:** decode the `World` root system's per-frame update — `World_UpdateBody` (advancing every
> active body), `World_OneShotEffect` (fire-and-forget effects), the active-body/effect lists they walk, and the
> `WorldAnim*` ambient-animation system — the tick that drives everything in the city.

The `World` is the root of the object graph ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)) — and each
frame, *it* is what advances the game. This chapter decodes the world update: how `World` walks its active-body
list (`World_UpdateBody`), fires one-shot effects (`World_OneShotEffect`), and ticks the ambient animations
(`WorldAnim*`). It's the *driver* of the frame — the loop that calls every body's simulation
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), every effect, every animation, once per tick.

> **Verified against the executable.** The world update is named in `speed.exe`: **`World`** (the root system,
> [C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)), **`World_UpdateBody`** (the per-body update),
> **`World_OneShotEffect`** (fire-and-forget effects), **`WorldUpdate`**, **`WorldBodyConn`** (the world-body
> connector), `WorldObject`, and the collision-event categories `World_Air`/`World_Civi`/`World_Flip`/`World_Spin`.
> The ambient-animation system is **`WorldAnim*`** (7) — `WorldAnimCtrl`, `WorldAnimEntity`, `WorldAnimEntityTree`,
> `WorldAnimInstanceEntry`, `WorldAnimTrigger`, `WorldAnimations`.

---

## Deep-dive pages

- [C64.1 — The world tick](01-world-tick.md): `World` and its per-frame update.
- [C64.2 — The active-body list](02-active-body-list.md): `World_UpdateBody` walking the bodies.
- [C64.3 — One-shot effects](03-one-shot-effects.md): `World_OneShotEffect` fire-and-forget.
- [C64.4 — World animations](04-world-animations.md): the `WorldAnim*` ambient system.
- [C64.5 — Reading the world update in RE](05-reading-world-update.md): navigating the tick.

---

## 64.1 The world tick

**`World`** ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)) is the root everything hangs off, and its
per-frame **`WorldUpdate`** ([C64.1](01-world-tick.md)) is the tick that advances the city — one of `FrameTick`'s
calls ([C37.4](../C37-Frame-Spine-Modules/04-frametick.md)). It walks the *active lists* — the bodies to simulate
(`World_UpdateBody`, [C64.2](02-active-body-list.md)) and the effects to play (`World_OneShotEffect`,
[C64.3](03-one-shot-effects.md)) — driving each. So `World` is the conductor of the frame's *content* (as
`FrameTick`, [Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md), is the conductor of the *engine
modules*).

## 64.2 The active-body list

**`World_UpdateBody`** ([C64.2](02-active-body-list.md)) walks the **active-body list** — every body that needs
simulating (the cars, [Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md); the dynamic props) — and
advances each ([C39.1](../C39-Vehicle-Simulation/01-pipeline.md)). Bodies *register* on this list when active and
*deregister* when inert (a slept smackable, [C43.5](../C43-Collision-Contacts/05-smackables.md), leaves it). So the
list is the *set of things being simulated* this frame, and `World_UpdateBody` is the loop that ticks them —
connected via `WorldBodyConn` ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)).

## 64.3 One-shot effects

**`World_OneShotEffect`** ([C64.3](03-one-shot-effects.md)) handles **fire-and-forget** effects — one-time visuals
that play once and vanish (an explosion, a spark burst, [Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)).
Unlike a persistent effect (a car's continuous exhaust, [C52.4](../C52-Effects-Particles/04-entity-effects.md)), a
one-shot is *spawned, played, and forgotten* — the world fires it and it self-completes. This is the pattern for
transient events (a crash's debris, a nitrous flash) — no owner to manage it, just a one-shot the world plays.

## 64.4 World animations

The **`WorldAnim*`** system ([C64.4](04-world-animations.md)) drives the world's *ambient animations*
([Chapter 26](../C26-World-Ambient-Animation/C26-World-Ambient-Animation.md)) — the moving, non-interactive scenery
that makes the city feel alive (flapping flags, swaying signs, working machinery). `WorldAnimEntityTree` organises
the animated entities spatially; `WorldAnimTrigger` fires animations by proximity/event; `WorldAnimCtrl` drives
them. These are ticked in the world update alongside the bodies and effects — the ambient life layer.

---

### Key takeaways

- **`World`** is the root system, and its **`WorldUpdate`** tick (one of `FrameTick`'s calls) advances the city's
  *content* each frame.
- **`World_UpdateBody`** walks the **active-body list** — every body being simulated (cars, dynamic props) —
  advancing each; bodies register/deregister as they become active/inert.
- **`World_OneShotEffect`** handles **fire-and-forget** effects — one-time visuals (explosions, bursts) spawned,
  played, and forgotten.
- **`WorldAnim*`** drives the **ambient animations** (flags, signs, machinery) — the non-interactive moving scenery
  that makes the city feel alive.
- `World` is the **conductor of the frame's content** — the counterpart to `FrameTick` (the conductor of the engine
  modules, [Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)).

**Next:** [Chapter 65 — The In-Race HUD & On-Screen Runtime](../C65-HUD-Runtime/C65-HUD-Runtime.md): the meters and
gauges of the race.
