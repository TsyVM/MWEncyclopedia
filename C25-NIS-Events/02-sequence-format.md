# C25.2 — The EventSequenceChunk Format

> **The one-sentence version:** the decoded CARP v26 script is a **registry + schedules** model — a registry of
> the entities and events the scene can reference, and schedules that place events on the timeline at specific
> times with parameters.

[← C25.1 — Event sequences are CARP scripts](01-carp-scripts.md) · [Chapter 25 hub](C25-NIS-Events.md) ·
[Next: C25.3 — The ENIS verb vocabulary →](03-enis-verbs.md)

---

## Registry + schedules

The `EventSequenceChunk` CARP blob (version 26) decodes into two complementary parts:

- **The registry** — a table of the **entities and event types** the script can reference: the cameras,
  characters, effects, and audio the scene involves, plus the `ENIS` verbs ([C25.3](03-enis-verbs.md)) it can
  fire. This is the script's *vocabulary* — what it's allowed to talk about.
- **The schedules** — the **timeline**: a list of entries, each "at time *t*, fire event *E* with parameters
  *P*." This is the script's *score* — what happens, and when.

Together they are a complete directorial script: the registry says *who and what*, the schedules say *when and
how*.

## The schedule is the timeline

The heart of the format is the schedule — a time-ordered list of events:

```
time     event (ENIS verb)        parameters
0.0      camera_cut               → camera A
0.5      play_animation           → character 1, anim "react"
1.2      trigger_effect           → fx "dust", at point P
1.2      play_audio               → NISAudio line 47
3.8      camera_cut               → camera B
…
```

Each entry binds a **time**, an **event** (an `ENIS` verb from the registry), and its **parameters** (which
entity, where, what). Playback ([C25.5](05-playback-assembly.md)) walks this schedule in time order, firing
each event as the clock reaches it — exactly like a MIDI sequence or a video-editing timeline, but for a
rendered cutscene.

## Why registry-plus-schedule

Separating a registry from schedules is the standard shape for any scripted timeline, and it buys:

- **Compact references.** The schedule refers to entities by a small index into the registry, not by repeating
  their full descriptions — the same indexing economy as scenery instances → infos
  ([C16.2](../C16-Scenery-Cull/02-models-instances.md)).
- **Validation.** Every event references registered entities/verbs, so a schedule entry that names something not
  in the registry is detectably invalid.
- **Reuse.** The registry can be shared across schedules; multiple timelines can direct the same cast.

It's the event-script analogue of the reflection vault's field/type/value model
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)): a schema of what's referable, plus data that
references it.

> ✅ *Verified (archive):* the `EventSequenceChunk` is decoded as CARP v26 with a registry + schedules model;
> the CARP container is verified directly ([C25.1](01-carp-scripts.md)).
> 🟡 *Reasoned:* the exact byte layout of each schedule entry (time/verb/param encoding) is the format's detail
> within the decoded registry+schedule model; the model and CARP container are verified.

## Reading the script

```python
def parse_event_sequence(carp):
    registry  = carp.table("registry")      # entities + event types (the vocabulary)
    schedules = carp.table("schedule")       # timed entries
    events = []
    for e in schedules:
        events.append({
            "time":   e.time,
            "verb":   registry.verb(e.event_id),      # ENIS verb (C25.3)
            "target": registry.entity(e.target_id),   # which camera/character/fx
            "params": e.params,
        })
    return sorted(events, key=lambda x: x["time"])     # time-ordered timeline
```

The output is a readable timeline — the cutscene's shot list, reconstructed from the file
([C25.5](05-playback-assembly.md)).

## Editing implications

- **Re-time by editing schedule entries.** Change an event's time to move a cut or cue earlier/later
  ([C25.6](06-editing-scripts.md)).
- **Re-direct by changing verbs/targets.** Point a camera cut at a different camera, or an animation event at a
  different character — via the registry index.
- **Keep references valid.** Every schedule entry must reference registered entities/verbs; adding an event may
  mean adding a registry entry.
- **Preserve time ordering sensibly.** The player walks the schedule in time; overlapping or out-of-order times
  should be intentional.

---

### Key takeaways

- The decoded script is **registry + schedules**: a vocabulary of entities/events, plus a timed list of events.
- The **schedule** is the timeline — each entry binds a time, an `ENIS` verb, a target, and parameters.
- Playback walks the schedule in time order, firing events as the clock reaches them (a cutscene timeline).
- Registry-plus-schedule gives compact references, validation, and reuse — the script analogue of the vault's
  schema+data.
- Edit by re-timing schedule entries and re-targeting verbs via the registry; keep references valid and timing
  intentional.

**Continue:** [C25.3 — The ENIS verb vocabulary](03-enis-verbs.md) · [Chapter 25 hub](C25-NIS-Events.md)
