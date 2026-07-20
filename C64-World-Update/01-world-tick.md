# C64.1 — The World Tick

> **The one-sentence version:** `World` is the root of the object graph, and its `WorldUpdate` tick — one of
> `FrameTick`'s calls — advances the city's content each frame by walking the active lists of bodies and effects.

[← Chapter 64 hub](C64-World-Update.md) · [Next: C64.2 — The active-body list →](02-active-body-list.md)

---

## World: the content conductor

The **`World`** ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)) is the root everything hangs off — the
sections ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)), the bodies
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)), the effects
([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)), the ambient animations
([C64.4](04-world-animations.md)). And each frame, **`WorldUpdate`** advances them all — it's the tick that drives
the game's *content*:

```
FrameTick (Ch 37) — the engine modules
   → WorldUpdate — the content:
      World_UpdateBody     — advance every active body (C64.2)
      World_OneShotEffect  — play the transient effects (C64.3)
      WorldAnim* update    — tick the ambient animations (C64.4)
```

So `World` is the *conductor of the content* — where `FrameTick`
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) conducts the *engine modules* (input, sim
driver, render, streaming), `WorldUpdate` conducts the *things in the world* (bodies, effects, animations).
`WorldUpdate` is one of `FrameTick`'s calls ([C37.4](../C37-Frame-Spine-Modules/04-frametick.md)) — the module that
advances the world.

> ✅ *Verified:* `World` ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)), `WorldUpdate`,
> `World_UpdateBody`, and `World_OneShotEffect` are present in `speed.exe` — the world tick and its sub-updates.

## The active lists

The key structure `WorldUpdate` walks is the **active lists** — the sets of things to advance
([C64.2](02-active-body-list.md)–[C64.3](03-one-shot-effects.md)):

- **The active-body list** — every body being *simulated* ([C64.2](02-active-body-list.md)): the cars, the dynamic
  props. `World_UpdateBody` walks it.
- **The one-shot-effect list** — the transient effects *playing* ([C64.3](03-one-shot-effects.md)): explosions,
  bursts. `World_OneShotEffect` walks it.
- **The animation list** — the ambient animations *running* ([C64.4](04-world-animations.md)).

So `WorldUpdate` is fundamentally a set of *list walks* — for each active body, tick it; for each one-shot effect,
advance it; for each animation, update it. This is the standard *update-list* pattern: register things on a list,
walk the list each frame to update them all. The lists are the world's *live content* — what's active *right now* —
and walking them is how the world advances. Things join a list when they become active
([C64.2](02-active-body-list.md)) and leave when inert, so the lists are always the *current* set.

## Why an active-list update

Driving the world through *active lists* ([above](#the-active-lists)) — rather than, say, scanning all objects — is
efficient and clean:

- **Only active things tick.** A slept smackable ([C43.5](../C43-Collision-Contacts/05-smackables.md)) or a
  finished effect isn't on the active list, so it costs *nothing* to update. Only what's *doing something* is
  ticked — the same pay-for-what's-active economy as traffic
  ([Chapter 61](../C61-Traffic-Ambient/C61-Traffic-Ambient.md)) and streaming
  ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).
- **Uniform update.** Every body ticks the same way ([C64.2](02-active-body-list.md)) — the list walk calls each
  body's update ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) polymorphically
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) — one loop, many body types.
- **Dynamic membership.** Things join/leave the lists as they activate/deactivate, so the update set is always
  current, no manual bookkeeping.

So the active-list update is the *engine of the frame's content* — a set of lists of live things, walked each tick.
It's the mechanism by which `World` ([above](#world-the-content-conductor)) advances everything: not a monolithic
"update the world" but a walk of the active bodies, effects, and animations. This is the frame's *content* half —
the counterpart to the engine-module half ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) —
and reading it shows how the world *moves* each frame.

## RE implications

- **`World`** is the content conductor — `WorldUpdate` (one of `FrameTick`'s calls) advances the world's content.
- **The active lists** — bodies (`World_UpdateBody`), one-shot effects (`World_OneShotEffect`), animations — are
  what it walks.
- **`WorldUpdate` is a set of list walks** — tick each active body, effect, animation.
- **Active-list update** — only active things tick (pay-for-what's-active), uniform, dynamic membership.

---

### Key takeaways

- **`World`** is the **content conductor** — its **`WorldUpdate`** tick (one of `FrameTick`'s calls) advances the
  world's *content* (bodies, effects, animations), where `FrameTick` conducts the *engine modules*.
- The tick walks the **active lists** — the active-body list (`World_UpdateBody`), the one-shot-effect list
  (`World_OneShotEffect`), the animation list.
- `WorldUpdate` is fundamentally a set of **list walks** — for each active thing, tick it — the standard
  update-list pattern.
- **Active-list update** is efficient — **only active things tick** (a slept prop costs nothing), uniform
  (polymorphic per body), and dynamically membered (join/leave as active/inert).
- It's the frame's **content half** — the counterpart to the engine-module half
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) — how the world moves each frame.

**Continue:** [C64.2 — The active-body list](02-active-body-list.md) · [Chapter 64 hub](C64-World-Update.md)
