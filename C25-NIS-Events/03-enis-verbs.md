# C25.3 — The ENIS Verb Vocabulary

> **The one-sentence version:** the events a schedule fires are `ENIS` verbs — the cutscene "action words" the
> engine understands (cut the camera, play an animation, trigger an effect, cue audio) — so a script reads like
> a shot list of typed directorial actions.

[← C25.2 — The EventSequenceChunk format](02-sequence-format.md) · [Chapter 25 hub](C25-NIS-Events.md) ·
[Next: C25.4 — NISAudio streams →](04-nisaudio.md)

---

## Verbs are the actions a cutscene can take

A schedule entry ([C25.2](02-sequence-format.md)) fires an **event**, and the events come from a fixed
vocabulary of **`ENIS` verbs** — the "NIS event" actions the engine knows how to perform. Each verb is a typed
directorial action; the schedule is a sequence of them over time. Reading the verbs turns the script from
opaque data into a legible shot list.

## The families of verb

The `ENIS` vocabulary covers what a director needs to stage a rendered scene:

- **Camera** — cut/move to a camera, set a shot, follow a target. The camera verbs are the backbone: a cutscene
  is largely a sequence of camera cuts over the action.
- **Animation** — play/stop a character animation ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)),
  drive a bone, pose a character. These trigger the skeletal animation on the cast.
- **Effect** — trigger a particle/visual effect (`fx*`, [C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md))
  at a place/time — sparks, dust, lights.
- **Audio** — cue a NISAudio line or sound ([C25.4](04-nisaudio.md)) — dialogue and scene sound.
- **Scene / flow** — start, end, wait, or branch the sequence; control the timeline itself.

Together these are enough to direct a cutscene: point the camera, move the characters, add effects, play the
lines, and control timing.

> 🟡 *Reasoned:* the `ENIS` verb *families* (camera, animation, effect, audio, scene control) are described from
> the vocabulary's role as a cutscene director's action set; the ✅ verified facts are the CARP
> `EventSequenceChunk` container ([C25.1](01-carp-scripts.md)) and the registry+schedule model
> ([C25.2](02-sequence-format.md)) that fire these verbs.

## A verb is a typed action with parameters

Each scheduled verb carries **parameters** appropriate to its type — the analogue of a function call:

```
camera_cut(camera_id)
play_animation(character_id, anim_id, blend)
trigger_effect(fx_id, position, orientation)
play_audio(nisaudio_id)
wait(duration)
```

So the schedule is effectively a **program**: a time-ordered list of verb calls with arguments, executed by the
playback engine ([C25.5](05-playback-assembly.md)). The registry ([C25.2](02-sequence-format.md)) supplies the
ids the parameters reference (which camera, which character, which effect), so a verb call is compact — a verb
plus small indices.

## Reading a script as a shot list

With the verbs named, a decoded schedule reads like a screenplay's shot list:

```
t=0.0  camera_cut(A)          — open on camera A
t=0.5  play_animation(cop, "point")   — cop points
t=1.2  play_audio(line_12)    — "Pull over!"
t=1.2  trigger_effect(dust)   — dust kicks up
t=3.8  camera_cut(B)          — cut to camera B
t=4.0  play_animation(player, "arrested")
…
```

This legibility is the payoff of decoding the vocabulary: you can *read* what a cutscene does, moment by
moment, and therefore reason about editing it ([C25.6](06-editing-scripts.md)).

## Editing implications

- **Change a verb to change the action.** Swap `camera_cut(A)` for `camera_cut(B)` to re-frame a moment; swap an
  animation id to change what a character does.
- **Adjust parameters.** Point a verb at a different camera/character/effect via its registry id.
- **Add verbs for new beats.** Insert a scheduled verb (with a registry entry if needed) to add a cut, effect,
  or line ([C25.2](02-sequence-format.md)).
- **Match verbs to available assets.** An animation verb needs an animation that exists
  ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)); an audio verb needs a NISAudio line
  ([C25.4](04-nisaudio.md)).

---

### Key takeaways

- Scheduled events are **`ENIS` verbs** — typed directorial actions (camera, animation, effect, audio, scene
  control).
- Camera verbs are the backbone; animation verbs drive the cast; effect/audio verbs add fx and dialogue.
- Each verb is a **call with parameters** referencing registry ids — the schedule is effectively a program.
- Decoded, a schedule reads like a **shot list**, moment by moment — the payoff of naming the verbs.
- Edit by changing verbs, adjusting parameters, or adding verbs — matched to available animations and audio.

**Continue:** [C25.4 — NISAudio streams](04-nisaudio.md) · [Chapter 25 hub](C25-NIS-Events.md)
