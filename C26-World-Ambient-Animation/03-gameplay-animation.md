# C26.3 — Gameplay Animation

> **The one-sentence version:** beyond ambience, gameplay objects animate through the same banks — driven by
> triggers and game state rather than a loop or a script — so animation ties into the systems that run the
> game.

[← C26.2 — World-ambient animation](02-world-ambient.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md) ·
[Next: C26.4 — Ambient vs cutscene playback →](04-ambient-vs-cutscene.md)

---

## Animation that reacts

World-ambient animation ([C26.2](02-world-ambient.md)) loops regardless of the game. **Gameplay animation** is
different: it plays *in response to gameplay* — a moving element that activates when a condition is met, a
mechanism that operates when triggered, an object that animates as part of an event. It uses the same
animation-bank format ([C26.1](01-one-format.md)), but its **driver is game state**, not a loop or a cutscene
script.

## Driven by triggers and state

Where a cutscene is driven by a scheduled timeline ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) and
ambience by a free-running loop ([C26.4](04-ambient-vs-cutscene.md)), gameplay animation is driven by the game
systems:

- **Triggers** ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)) — entering a region can start
  an animation (a gate opens, a mechanism moves).
- **Events** — a gameplay event ([C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md)) can fire
  an animation as part of its reaction.
- **State** — an object's animation reflects a game state (active/inactive, progress).

So gameplay animation sits at the junction of the animation format and the gameplay systems: the format
provides the *motion*, the systems provide the *cue*.

## Same format, gameplay clock

The bank is evaluated exactly as in Chapter 24 — skeleton + keyframes — but its clock is advanced by the
gameplay condition rather than a loop or schedule:

```
gameplay condition (trigger / event / state)
   → start / advance / stop the animation bank's clock
   → skeleton posed by keyframes at the current time (C24)
   → the object animates in response to the game
```

This is the third driver in the "one format, many users" model ([C26.1](01-one-format.md)): the cue comes from
gameplay, so the animation is reactive — it happens because *the game made it happen*, at the moment it's
needed.

> 🟡 *Reasoned:* gameplay animation as a trigger/state-driven use of the verified animation-bank format is
> inferred from the format's generality and the gameplay systems; the animation format is verified
> ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)) and the trigger/event systems are decoded
> ([Chapters 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md), [17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)).

## Where gameplay animation connects

Gameplay animation is a connective tissue between systems already decoded in the book:

- **Triggers** ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)) supply the *when* (enter a
  region → animate).
- **The vault** ([Chapters 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)–[14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md))
  supplies gameplay *parameters* that may gate or tune the animation.
- **The animation bank** (this format) supplies the *motion*.

So decoding gameplay animation is mostly about tracing the cue: which trigger/event/state starts which bank on
which object. The bank itself is the familiar Chapter 24 format.

## What you can recover

As with every user of the format:

- **The rig** — the gameplay object's skeleton, from the ELF symbols
  ([C24.3](../C24-NIS-Animation/03-skeleton.md)) — recoverable.
- **The motion** — the keyframes — the frontier ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)).
- **The cue** — decodable via the trigger/event systems ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)) —
  you can determine *what starts* the animation even if the motion data is opaque.

So you can map *what triggers what* and recover the rigs, while the quantised motion remains preserve-raw
([C26.5](05-shared-frontier.md)).

---

### Key takeaways

- **Gameplay animation** uses the animation-bank format but is driven by **game state** — triggers, events,
  status.
- Its clock is advanced by a gameplay condition, not a loop (ambient) or schedule (cutscene) — reactive motion.
- It connects the animation format to the trigger ([Ch 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md))
  and vault ([Ch 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)) systems.
- Decoding it is tracing the cue (which trigger/event/state → which bank) plus the Chapter 24 format.
- Recover the rig and the cue; the keyframed motion stays the shared frontier.

**Continue:** [C26.4 — Ambient vs cutscene playback](04-ambient-vs-cutscene.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md)
