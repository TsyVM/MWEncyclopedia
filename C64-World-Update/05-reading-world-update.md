# C64.5 — Reading the World Update in RE

> **The one-sentence version:** navigate the world update by `World`/`WorldUpdate`, `World_UpdateBody` (the
> active-body list), `World_OneShotEffect` (fire-and-forget), and `WorldAnim*` — reading the tick as a walk of the
> active lists of bodies, effects, and animations.

[← C64.4 — World animations](04-world-animations.md) · [Chapter 64 hub](C64-World-Update.md) ·
[Next: Chapter 65 — The In-Race HUD & On-Screen Runtime →](../C65-HUD-Runtime/C65-HUD-Runtime.md)

---

## Anchors for world-update RE

The world update is anchored on verified strings:

- **`World`** / `WorldUpdate` — the root and its tick ([C64.1](01-world-tick.md)).
- **`World_UpdateBody`** / `WorldBodyConn` — the active-body list ([C64.2](02-active-body-list.md)).
- **`World_OneShotEffect`** — fire-and-forget effects ([C64.3](03-one-shot-effects.md)).
- **`WorldAnim*`** — the ambient animations ([C64.4](04-world-animations.md)).

From these, the world update is navigable: the tick, the bodies, the effects, and the animations.

## The RE workflow

Reading the world update:

1. **Find the tick** — `World`/`WorldUpdate` ([C64.1](01-world-tick.md)); one of `FrameTick`'s calls.
2. **Trace the body update** — `World_UpdateBody` and the active-body list ([C64.2](02-active-body-list.md)).
3. **Trace the effects** — `World_OneShotEffect` ([C64.3](03-one-shot-effects.md)).
4. **Find the animations** — `WorldAnim*` ([C64.4](04-world-animations.md)).

The output is the full world-update picture: the tick and its three content sub-updates.

## The world update is the frame's content half

The organising insight is that the world update is the *content half* of the frame
([C64.1](01-world-tick.md)) — complementary to the engine-module half
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)):

```
FrameTick (Ch 37) — the ENGINE:
   input → sim driver → streaming → render → present
      ↕ (the sim driver / WorldUpdate call)
WorldUpdate (this chapter) — the CONTENT:
   World_UpdateBody     (the bodies — Ch 39)
   World_OneShotEffect  (the effects — Ch 52)
   WorldAnim* update    (the animations — Ch 26)
```

So `FrameTick` ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) is the *machine* running the
frame (the modules), and `WorldUpdate` is the *content* the machine advances (the bodies, effects, animations).
Together they are the complete frame: the engine ticks its modules
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)), one of which advances the world's content
(this chapter). Reading the world update completes the *frame* picture — Chapter 37 showed the module spine, this
chapter shows the content it drives. The two together are how *everything* advances each frame: the engine and the
world it runs.

## The active lists are the live game state

A practical RE payoff ([C64.2](02-active-body-list.md)): the *active lists* are the *live game state* — reading
them (in a dump) tells you everything happening *now*:

- **The active-body list** ([C64.2](02-active-body-list.md)) — every body simulating (the physics population).
- **The one-shot-effect list** ([C64.3](03-one-shot-effects.md)) — every transient effect playing.
- **The animation list** ([C64.4](04-world-animations.md)) — every ambient animation running.

So the world's *dynamic* state is these lists — walk them and you have the complete live content
([C38.7](../C38-Resource-Streaming-Residency/07-reading-streaming.md)). Combined with the streaming manager's
resident list ([C38.2](../C38-Resource-Streaming-Residency/02-sections-residency.md), what's *loaded*) and the class
families ([C32.6](../C32-Runtime-Class-System/06-reading-binary.md), all *objects*), the active lists complete the
*live-state* picture: what's loaded (streaming), what exists (class families), and what's *active* (the world
update's lists). Reading the world update is thus reading the *pulse* of the running game — the set of things being
advanced each frame.

## RE implications

- **Anchor on** `World`/`WorldUpdate`, `World_UpdateBody`, `World_OneShotEffect`, and `WorldAnim*`.
- **The RE workflow** — the tick → body update → effects → animations.
- **The world update is the frame's content half** — complementary to the engine-module spine (Ch 37).
- **The active lists are the live game state** — bodies, effects, animations being advanced now.

---

### Key takeaways

- The world update is anchored on **`World`/`WorldUpdate`**, **`World_UpdateBody`**, **`World_OneShotEffect`**, and
  **`WorldAnim*`**.
- The RE workflow: **the tick → body update → effects → animations**.
- The world update is the **frame's content half** — `FrameTick` runs the *engine modules*, `WorldUpdate` advances
  the *content* (bodies, effects, animations) — together, the complete frame.
- The **active lists are the live game state** — the active-body list (physics population), the one-shot-effect
  list, the animation list — reading them gives everything happening *now*.
- Combined with streaming (what's *loaded*) and the class families (what *exists*), the active lists complete the
  **live-state picture** — reading the world update is reading the **pulse** of the running game.

**Next:** [Chapter 65 — The In-Race HUD & On-Screen Runtime](../C65-HUD-Runtime/C65-HUD-Runtime.md): the meters and
gauges of the race.

**Sources:** `speed.exe` (verified: `World` [C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md); `WorldUpdate`;
`World_UpdateBody`; `World_OneShotEffect`/`OneShotEffect`; `WorldBodyConn`; `WorldObject`; `World_Air`/`World_Civi`/
`World_Flip`/`World_Spin`; `WorldAnimations`/`WorldAnimEntity`/`WorldAnimEntityTree`/`WorldAnimInstanceEntry`/
`WorldAnimTrigger`/`WorldAnimCtrl`).
