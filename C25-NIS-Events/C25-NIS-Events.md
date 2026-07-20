# Chapter 25 — NIS Event Timelines, Scripts & Playback

> **Goal of this chapter:** decode the *direction* of a cutscene — the CARP event-sequence script that
> schedules the `ENIS` verbs over time — and see how the player assembles a finished scene from three sources:
> the animation, the event script, and the NIS audio streams.

Chapter 24 gave you a cutscene's *actors* — the skeletons and (opaque) keyframes. This chapter gives you its
*director*: the **event sequence** that says *what happens when* — cut the camera here, trigger this effect
there, play this line now. It is a **CARP** script of scheduled `ENIS` events, and it is the decoded,
editable side of NIS. Put it together with the animation and the audio, and you have a playable scene.

> **Verified against retail data.** The `EventSequenceChunk` (`0x0003B811`) in `TRACKS/L2RA.BUN` is, after an
> 8-byte `0x11` sentinel, a **CARP** blob (magic `PRAC`) — the same attribute-blob format as the road network
> ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)). The NIS scene bundle likewise contains
> CARP event data (`PRAC` at `0x15B7D0`). The NIS audio ships as `NISAudio.big`/`.csi`/`.evt`/`.idx`, with the
> `.csi` index carrying a `MOIR` magic.

---

## Deep-dive pages

- [C25.1 — Event sequences are CARP scripts](01-carp-scripts.md): the `EventSequenceChunk` as a CARP
  attribute blob, not a chunk tree.
- [C25.2 — The EventSequenceChunk format](02-sequence-format.md): the registry + schedules model of a decoded
  CARP v26 script.
- [C25.3 — The ENIS verb vocabulary](03-enis-verbs.md): the events a script can schedule.
- [C25.4 — NISAudio streams](04-nisaudio.md): the `.big`/`.csi`/`.evt`/`.idx` audio and its index.
- [C25.5 — Three-source playback assembly](05-playback-assembly.md): animation + script + audio → a scene.
- [C25.6 — Editing cutscene scripts](06-editing-scripts.md): re-timing and re-directing a cutscene from data.

---

## 25.1 The script is CARP

A cutscene's timeline is an **event sequence** stored as a **CARP** attribute blob — the `EventSequenceChunk`
(`0x0003B811`), wrapped by an `EventSequencePack` (`0x8003B810`). Verified: after the 8-byte `0x11` sentinel,
the chunk carries the `CARP` magic (`PRAC` reversed), exactly like the road network
([C18.1](../C18-Road-Network-CARP/01-carp-format.md)). So the same rule applies: it is **attribute data, not a
chunk tree** — parse it as a CARP tag directory, not with the universal walker ([C25.1](01-carp-scripts.md)).

## 25.2 Registry + schedules

The decoded `EventSequenceChunk` (CARP v26) is a **registry + schedules** model: a registry of the entities
and events the scene can reference, and **schedules** that place events on the timeline — "at time *t*, fire
event *E* with these parameters" ([C25.2](02-sequence-format.md)). The registry is the vocabulary; the
schedules are the score. Reading them gives you the cutscene's full direction: every camera cut, effect, and
cue, with its timing.

## 25.3 The ENIS verbs

The events a schedule fires come from the **`ENIS` verb vocabulary** — the set of cutscene "verbs" the engine
understands: move/cut the camera, play an animation, trigger an effect, play audio, and so on
([C25.3](03-enis-verbs.md)). Each scheduled entry names an `ENIS` verb and its arguments, so the script reads
like a shot list: a sequence of typed directorial actions on the timeline.

## 25.4 The audio streams

A cutscene's dialogue and sound are the **NISAudio** streams — `NISAudio.big` (the audio payload), `.csi` (an
index, magic `MOIR`), `.evt` (events), and `.idx` (index). The event script cues these at the right moments, so
a line of dialogue plays on the frame the schedule specifies ([C25.4](04-nisaudio.md)). The audio itself is the
shared codec layer ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)).

## 25.5 Three sources become a scene

Playback **assembles three sources** in lockstep ([C25.5](05-playback-assembly.md)):

```
animation (Chapter 24: skeleton + keyframes)  ─┐
event script (this chapter: CARP + ENIS verbs) ─┼─▶  the played cutscene
NIS audio (NISAudio streams)                   ─┘
```

The script is the **conductor**: it advances the timeline, drives the camera and effects, triggers the
character animation, and cues the audio — so the three sources play as one directed scene. Understanding this
assembly is understanding how a NIS cutscene actually runs.

---

### Key takeaways

- A cutscene's timeline is an **`EventSequenceChunk`** — a **CARP** attribute blob (verified `PRAC` magic), not
  a chunk tree.
- The decoded format is **registry + schedules**: a vocabulary of entities/events plus timed event placements.
- Scheduled events are **`ENIS` verbs** — camera, animation, effect, and audio actions, like a shot list.
- Cutscene audio is the **NISAudio** streams (`.big`/`.csi`(`MOIR`)/`.evt`/`.idx`), cued by the script.
- Playback **assembles three sources** — animation + event script + audio — with the script as conductor.

**Next:** Part V is complete. The world (Part IV), audio/video/animation (Part V), and their editing
disciplines are now decoded end to end.
