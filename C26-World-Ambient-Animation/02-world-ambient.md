# C26.2 — World-Ambient Animation

> **The one-sentence version:** parts of the city move — dockside cranes swing, harbour ships drift, a blimp
> crosses the sky — as world-ambient animations: animation banks attached to world objects and looped
> continuously to make the world feel alive.

[← C26.1 — One format, many users](01-one-format.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md) ·
[Next: C26.3 — Gameplay animation →](03-gameplay-animation.md)

---

## The living city

A convincing open world isn't only static geometry ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) —
it has **motion in the background**. Most Wanted's Rockport moves in the periphery: **cranes** swing loads at
the docks, **ships** drift in the harbour, a **blimp** crosses overhead. These are **world-ambient
animations** — animation banks ([C26.1](01-one-format.md)) bound to world objects and played continuously,
independent of the player. They are what separate a lived-in city from a diorama.

## Animated scenery

Think of world-ambient animation as the animated tier of scenery:

- **Static scenery** ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) — buildings, signs, props that
  don't move; a `SceneryInstance` with a fixed transform.
- **Ambient-animated objects** — cranes, ships, the blimp; a world object with an **animation bank** that
  moves its rig over time.

The crane has a skeleton (base, arm, hook — [C24.3](../C24-NIS-Animation/03-skeleton.md)) and a keyframed loop
that swings it; the ship has a rig and a gentle drift; the blimp a slow traverse. Each is the animation format
of Chapter 24 applied to a world object rather than a cutscene character.

## They loop, undirected

The defining trait of ambient animation is that it's **continuous and undirected** ([C26.4](04-ambient-vs-cutscene.md)):

- **No script.** Unlike a cutscene ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)), nothing schedules the
  crane's motion — it just loops.
- **No player dependence.** It plays whether or not you're watching (subject to streaming/visibility —
  [Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)); the world moves on its own.
- **Endless.** The keyframe loop repeats, so the crane swings forever, the blimp circles, the harbour lives.

This is the cheapest way to animate the background: one looping bank per object, evaluated when the object is
resident and visible.

> 🟡 *Reasoned:* the specific ambient objects (cranes, ships, blimp) and their loop-driven playback are the
> world-ambient use of the verified animation-bank format ([C26.1](01-one-format.md)); the format itself is
> verified ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)).

## Ambience within the streaming world

Ambient animations live inside the streamed world, so they obey its rules
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)):

- **Resident when near.** An ambient object animates when its section is resident; a crane far across the map
  isn't consuming animation time.
- **Culled when unseen.** Visibility ([C15.5](../C15-Track-Streaming/05-visibility.md),
  [C16.5](../C16-Scenery-Cull/05-cull-tree.md)) skips animating/drawing objects you can't see.
- **Placed like scenery.** The animated object sits in the world at a transform, like an instance, but carries
  a bank instead of (or alongside) a static mesh.

So ambient animation is a modest, bounded cost: only the nearby, visible movers animate, which is what lets a
whole city's worth of ambience run cheaply.

## What you can recover

Applying Chapter 24's decoding to an ambient object:

- **The rig** — the crane/ship/blimp skeleton from the ELF symbol table
  ([C24.3](../C24-NIS-Animation/03-skeleton.md)) — fully recoverable.
- **The motion** — the keyframed loop in the bank — still the frontier
  ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md), [C26.5](05-shared-frontier.md)).

So you can identify and rebuild the ambient objects' rigs, but not yet reliably reproduce their motion from the
banks — the same split as cutscenes.

---

### Key takeaways

- World-ambient animation makes the city move: cranes, ships, and the blimp are looped animation banks on world
  objects.
- It's the **animated tier of scenery** — a rig + keyframe loop instead of a fixed transform.
- Ambient motion is **continuous, undirected, endless** — no script, no player dependence.
- It obeys streaming/visibility: only nearby, visible movers animate, keeping the cost bounded.
- You can recover the ambient rigs (skeletons); their keyframed motion remains the shared frontier.

**Continue:** [C26.3 — Gameplay animation](03-gameplay-animation.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md)
