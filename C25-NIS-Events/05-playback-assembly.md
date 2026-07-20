# C25.5 — Three-Source Playback Assembly

> **The one-sentence version:** playing a NIS cutscene assembles three sources in lockstep — the animation
> (skeleton + keyframes), the event script (CARP + `ENIS` verbs), and the NIS audio — with the event script as
> the conductor advancing the timeline and cueing the other two.

[← C25.4 — NISAudio streams](04-nisaudio.md) · [Chapter 25 hub](C25-NIS-Events.md) ·
[Next: C25.6 — Editing cutscene scripts →](06-editing-scripts.md)

---

## Three sources, one scene

A finished cutscene is not one file — it is the **assembly** of three:

```
animation   (Chapter 24: skeletons + keyframes)   — the characters' motion
event script (this chapter: CARP + ENIS verbs)     — the direction (camera, timing, cues)
NIS audio    (NISAudio streams, C25.4)             — the dialogue and sound
                              │
                              ▼
                      the played cutscene
```

Each source is decoded in its own chapter; playback is where they come together. The scene you watch is these
three, synchronised on a shared clock.

## The script is the conductor

The event script ([C25.2](02-sequence-format.md)) is the component that *drives* the others. It owns the
timeline, and as its clock advances it fires `ENIS` verbs ([C25.3](03-enis-verbs.md)) that command the other
two sources:

- **Camera verbs** set the shot — the view onto the scene.
- **Animation verbs** trigger the character animation ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)) —
  telling the skeletons which motions to play, when.
- **Audio verbs** cue the NISAudio streams ([C25.4](04-nisaudio.md)) — landing dialogue and sound on the right
  frames.
- **Effect verbs** fire particle/visual effects at scheduled moments.

So the script is the conductor and the animation and audio are the instruments: none of them decides the timing;
the schedule does. This is why the event sequence is the *editable* heart of a cutscene — change the script and
you change the whole performance ([C25.6](06-editing-scripts.md)).

## The playback loop

Assembly runs a timeline loop:

```python
def play_cutscene(script, animations, audio):
    clock = 0.0
    while not script.done(clock):
        for event in script.due_events(clock):        # verbs whose time has arrived (C25.2)
            if event.verb == "camera_cut":   set_camera(event.target)
            elif event.verb == "play_animation": start_anim(animations[event.target], event.params)
            elif event.verb == "play_audio":  cue_audio(audio[event.target])
            elif event.verb == "trigger_effect": spawn_fx(event.target, event.params)
        render_frame(active_camera, posed_characters(animations, clock), active_effects)
        mix_audio(active_audio, clock)
        clock += dt
```

Each frame: fire the verbs due at this time, pose the characters from their running animations, render through
the active camera, and mix the cued audio. The three sources advance together on `clock`, which is why they
stay in sync — they share one timeline, driven by the script.

> ✅ *Verified:* the three sources exist and are decoded to the extent stated — animation (skeleton verified,
> keyframes ⏳ [C24.5](../C24-NIS-Animation/05-keyframe-problem.md)), event script (CARP verified,
> [C25.1](01-carp-scripts.md)), NIS audio ([C25.4](04-nisaudio.md)); the script-as-conductor assembly is the
> playback model.
> 🟡 *Reasoned:* the exact per-frame playback loop is described from the registry+schedule model and the three
> sources; the sources and the CARP script container are verified.

## The frontier shows here

The assembly makes the state of knowledge concrete. Two of the three sources are decoded and editable:

- **Event script** — decoded (CARP registry + schedules); you can read and re-time the whole direction
  ([C25.6](06-editing-scripts.md)).
- **NIS audio** — decoded (shared codecs); you can extract and replace dialogue ([C25.4](04-nisaudio.md)).
- **Character animation** — the *skeleton* is decoded, but the *keyframes* are not
  ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)).

So you can fully reconstruct a cutscene's **direction and audio**, and its **cast and rig**, but not yet its
**character motion** from the files alone — and on PC that motion is static anyway
([C24.6](../C24-NIS-Animation/06-ps2-vs-pc.md)). The assembly is the clearest place to see exactly what's open.

## Editing implications

- **Re-direct via the script.** Because the script conducts, editing it ([C25.6](06-editing-scripts.md)) re-times
  cuts, changes cameras, and re-cues audio — the highest-leverage cutscene edit.
- **Replace audio independently.** Swap dialogue in NISAudio ([C25.4](04-nisaudio.md)) and re-time cues if
  lengths change.
- **Animation is the constrained source.** You can trigger *existing* animations differently via the script, but
  authoring *new* character motion awaits the keyframe decode
  ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)).
- **Keep the three in sync.** The shared clock is the contract — a change to one source's timing must respect the
  others.

---

### Key takeaways

- A cutscene is the **assembly of three sources**: animation, event script, and NIS audio, on a shared clock.
- The **event script is the conductor** — it fires `ENIS` verbs that command the camera, animation, audio, and
  effects at scheduled times.
- The playback loop fires due verbs, poses characters, renders through the active camera, and mixes cued audio,
  each frame.
- The assembly shows the frontier: **script and audio decoded**, **skeleton decoded**, **keyframes ⏳** (and
  static on PC).
- Re-direct via the script (highest leverage), replace audio independently; new character motion awaits the
  keyframe decode.

**Continue:** [C25.6 — Editing cutscene scripts](06-editing-scripts.md) · [Chapter 25 hub](C25-NIS-Events.md)
