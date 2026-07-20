# C25.6 — Editing Cutscene Scripts

> **The one-sentence version:** because the event script conducts the cutscene, editing it is the
> highest-leverage change — re-time cuts, swap cameras, re-cue audio, and add beats by editing the CARP
> registry and schedules — all without touching the (undecoded) character keyframes.

[← C25.5 — Three-source playback assembly](05-playback-assembly.md) · [Chapter 25 hub](C25-NIS-Events.md) ·
[Next: Chapter 26 — World-Ambient & Gameplay Animation Banks →](../README.md)

---

## The script is the editable heart

The cutscene's direction lives in the event script ([C25.2](02-sequence-format.md)), and the script *conducts*
the scene ([C25.5](05-playback-assembly.md)) — so editing it changes the performance without needing the
undecoded keyframes ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)). This is the practical takeaway of
the whole chapter: **you can re-direct a NIS cutscene from data**, because the direction is decoded even though
the character motion isn't.

## What you can edit

Working within the CARP registry + schedules ([C25.2](02-sequence-format.md)):

- **Re-time events.** Change a schedule entry's time to move a camera cut, an effect, or an audio cue
  earlier/later — retiming the whole performance.
- **Re-frame with cameras.** Point a `camera_cut` at a different camera in the registry
  ([C25.3](03-enis-verbs.md)) to change the shot.
- **Re-cue audio.** Change which NISAudio stream an `audio` verb fires, or when ([C25.4](04-nisaudio.md)).
- **Re-trigger animations.** Fire a *different existing* animation on a character (you can't author new motion,
  but you can re-sequence what exists).
- **Add or remove beats.** Insert a scheduled verb (with a registry entry if it references something new) for a
  new cut/effect/line, or delete one.

```python
def retime_event(script, event_index, new_time):
    script.schedule[event_index].time = new_time     # move a cut/cue on the timeline
    script.schedule.sort(key=lambda e: e.time)       # keep time order (C25.2)

def reframe_cut(script, event_index, new_camera_id):
    ev = script.schedule[event_index]
    assert ev.verb == "camera_cut" and new_camera_id in script.registry.cameras
    ev.target_id = new_camera_id                     # point at a registered camera
```

## The rules

Script edits obey the CARP discipline ([C25.1](01-carp-scripts.md)) and the registry+schedule model:

- **Parse and write CARP correctly.** Branch it out of the chunk walker; read tags reversed; don't corrupt the
  blob ([C18.1](../C18-Road-Network-CARP/01-carp-format.md)).
- **Keep references valid.** Every schedule entry must reference a registered entity/verb; adding an event that
  names something new means adding it to the registry ([C25.2](02-sequence-format.md)).
- **Respect the shared clock.** The animation and audio sync to the schedule's timeline
  ([C25.5](05-playback-assembly.md)); re-timing one cue may require adjusting others so the scene stays
  coherent.
- **Match verbs to assets.** An animation verb needs an animation that exists
  ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)); an audio verb a NISAudio stream that exists
  ([C25.4](04-nisaudio.md)).
- **Repack the CARP/pack.** Adding/removing entries resizes the `EventSequenceChunk`; fix the
  `EventSequencePack` wrapper size and any container above it ([C25.1](01-carp-scripts.md)).

## What you can't (yet) do

Be clear about the boundary:

- **New character motion.** Authoring fresh keyframed animation awaits the keyframe-quantisation decode
  ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)) — you can re-sequence existing animations, not create
  new ones.
- **PC playback of motion.** NIS character animation is static on PC ([C24.6](../C24-NIS-Animation/06-ps2-vs-pc.md)),
  so script-driven re-sequencing of character motion shows fully on PS2, not PC.

So script editing is powerful for *direction, timing, cameras, effects, and audio* — the bulk of a cutscene's
feel — and constrained only where it depends on the undecoded motion.

## Verify

After editing a script:

1. **CARP parses** — the `EventSequenceChunk` reads back as a valid CARP registry + schedules
   ([C25.1](01-carp-scripts.md)).
2. **References resolve** — every schedule entry's verb/target is registered.
3. **Sizes fixed** — the pack/wrapper sizes match after any resize.
4. **Play it in context** — on the platform that shows the motion (PS2), watch the cutscene: cuts land, audio
   syncs, effects fire on time. This is the decisive test — a timeline is only right when it plays right.

---

### Key takeaways

- The event script **conducts** the cutscene, so editing it re-directs the scene without the undecoded
  keyframes.
- You can re-time events, re-frame cameras, re-cue audio, re-trigger existing animations, and add/remove beats.
- Follow CARP discipline (branch out of the walker, tags reversed), keep registry references valid, respect the
  shared clock, and match verbs to real assets.
- You **can't** author new character motion (keyframes ⏳) — only re-sequence existing animation; motion is
  static on PC.
- Verify CARP parse, reference resolution, sizes, and — decisively — play the cutscene in context.

**Continue:** [Chapter 26 — World-Ambient & Gameplay Animation Banks](../README.md) · [Chapter 25 hub](C25-NIS-Events.md)
