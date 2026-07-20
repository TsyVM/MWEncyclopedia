# C26.4 — Ambient vs Cutscene Playback

> **The one-sentence version:** the same animation bank plays three ways — a cutscene's scripted one-shot
> timeline, an ambient loop that never stops, and a gameplay clip cued by state — differing only in the clock
> that advances it.

[← C26.3 — Gameplay animation](03-gameplay-animation.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md) ·
[Next: C26.5 — The shared frontier →](05-shared-frontier.md)

---

## Three clocks, one format

The animation bank ([C26.1](01-one-format.md)) is a skeleton + keyframes with no opinion about *how* it plays.
The playback *driver* supplies the clock, and the three users drive it differently:

| | Cutscene | World-ambient | Gameplay |
|---|---|---|---|
| Driver | scripted timeline ([Ch 25](../C25-NIS-Events/C25-NIS-Events.md)) | free-running loop | game state / trigger |
| Lifetime | one-shot | continuous | event-scoped |
| Looping | usually no | yes, endlessly | sometimes |
| Cue | the schedule | none (always on) | a gameplay condition |
| Directed? | yes (camera, timing) | no | partly |

So "playing an animation" means "advancing its clock," and the three users differ in *what advances it* and
*for how long*.

## Cutscene playback: directed and finite

A cutscene ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) plays a bank as part of a **directed timeline**:
an `ENIS` verb starts the animation at a scheduled time, it plays through (possibly blended with others), and
the scene moves on. It's **finite** (the shot ends) and **directed** (the camera and timing are scripted). The
animation serves the director.

## Ambient playback: looping and free

Ambient playback ([C26.2](02-world-ambient.md)) is the opposite: the bank's clock **loops** endlessly and
**freely** — no schedule, no director, no end. The crane swings, the loop repeats, forever (while resident and
visible). The only control is streaming/visibility deciding *whether* to evaluate it, not *when* it plays. It's
the simplest driver: advance the clock, wrap at the loop end, repeat.

## Gameplay playback: cued and scoped

Gameplay playback ([C26.3](03-gameplay-animation.md)) sits between: the clock is advanced **when a gameplay
condition holds** — a trigger fires it, it plays for the event's scope, and it stops when the condition ends.
It may loop while active (a running mechanism) or play once (a one-shot reaction). The cue is gameplay; the
lifetime is the event.

## Why one format supports all three

That a single format serves scripted, looping, and cued playback is the strength of separating *animation data*
from *animation driver*:

- **The bank is inert data** — skeleton + keyframes, no playback policy baked in.
- **The driver supplies policy** — one-shot, loop, or cue.
- **The evaluator is shared** — the same skeleton+keyframe evaluation ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md))
  runs whichever driver feeds it a time.

This is the same data/driver separation as the GIN engine synth (grains + RPM driver,
[C22.4](../C22-Engine-Sound-GIN/04-rpm-bridge.md)) and the music (sections + MPF director,
[C21.4](../C21-Music-MUS-MPF/04-mpf-director.md)): inert content, a driver that decides timing, and a shared
evaluator.

> 🟡 *Reasoned:* the three playback drivers (script/loop/cue) are the observed uses of the shared, verified
> animation format; the format and the cutscene driver ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) are
> verified.

## Editing implications

- **Identify the driver first.** To change how an animation behaves, find whether it's script-, loop-, or
  cue-driven — the format is the same, the control differs.
- **Ambient loops are edited at the loop.** Change the loop range/rate to alter continuous motion
  ([C26.2](02-world-ambient.md)).
- **Cutscene timing is edited in the script** ([C25.6](../C25-NIS-Events/06-editing-scripts.md)).
- **Gameplay cues are edited in the triggering system** ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)).

---

### Key takeaways

- The same bank plays three ways — cutscene (scripted one-shot), ambient (endless loop), gameplay (cued/scoped)
  — differing only in the clock driving it.
- Cutscene = directed + finite; ambient = free + looping; gameplay = cued + event-scoped.
- One format supports all three because animation **data** is separate from the **driver** and a shared
  evaluator.
- It's the same data/driver split as the GIN synth and the music director.
- Edit the *driver* for behaviour (loop range, script timing, trigger cue), not the shared format.

**Continue:** [C26.5 — The shared frontier](05-shared-frontier.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md)
