# C25.4 — NISAudio Streams

> **The one-sentence version:** a cutscene's dialogue and sound are the NISAudio streams — `NISAudio.big` (the
> audio payload), `.csi` (a `MOIR` index), `.evt` (events), and `.idx` (index) — cued by the event script and
> decoded with the shared codec layer.

[← C25.3 — The ENIS verb vocabulary](03-enis-verbs.md) · [Chapter 25 hub](C25-NIS-Events.md) ·
[Next: C25.5 — Three-source playback assembly →](05-playback-assembly.md)

---

## The NISAudio file set

Cutscene audio ships as a small family of files in `SOUND/STREAMS/`, verified on the retail data:

| File | Magic / role |
|---|---|
| `NISAudio.big` | the **audio payload** — the concatenated dialogue/sound streams |
| `NISAudio.csi` | an **index** (magic `MOIR`) — locates streams in the payload |
| `NISAudio.evt` | **events** — audio cue/timing data |
| `NISAudio.idx` | an **index** — stream directory |

This is the familiar **routing-plus-payload** split seen throughout the audio system
([C19.2](../C19-Audio-Banks/02-snr-spt.md)): the `.big` holds the audio bytes, and the `.csi`/`.idx`/`.evt`
files index and cue them. A cutscene's audio verb ([C25.3](03-enis-verbs.md)) names a stream, the index locates
it in the `.big`, and the codec layer decodes it.

## The audio is the shared codec layer

The audio inside `NISAudio.big` is the same **EA-ADPCM (XAS)/PCM** as everything else
([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)) — no new codec. So decoding cutscene dialogue reuses
the decoder you built for banks, music, and movies ([C20.6](../C20-Audio-Codecs/06-portable-decoder.md)):
locate the stream via the index, slice the `.big`, decode with EA-XAS. The NISAudio set is a *container* around
the shared codecs, like every other audio container.

## Cued by the script

The event script ([C25.2](02-sequence-format.md)) is what makes the audio a *cutscene*: an `audio` verb at a
scheduled time names a NISAudio stream, and the player cues it so the line plays on the intended frame. So the
`.evt` events and the schedule work together to time dialogue against the action — a character's line lands as
they speak, an effect's sound fires with the effect. The script is the conductor; NISAudio is one of the
instruments ([C25.5](05-playback-assembly.md)).

> ✅ *Verified:* the NISAudio set is `NISAudio.big`/`.csi`/`.evt`/`.idx`, with `.csi` carrying a `MOIR` magic —
> a routing/index-plus-payload audio container.
> 🟡 *Reasoned:* the exact `MOIR` index record layout is the format's detail; the file set, its
> routing/payload structure, and its shared-codec audio are verified/consistent with the audio system.

## Why a separate audio set for cutscenes

NISAudio is its own set (rather than reusing the bank system) because cutscene audio has different needs:

- **Streamed, not resident.** Cutscene dialogue is long and played linearly, so it streams from the `.big`
  ([like the world](../C15-Track-Streaming/03-residency.md)) rather than living in a resident bank.
- **Timeline-cued.** It's fired by the event schedule at precise times, so it needs event/index files (`.evt`,
  `.idx`) tying streams to cues, not just an id lookup.
- **Scene-scoped.** A cutscene's audio is a coherent set for that scene, packaged together.

So NISAudio is the cutscene-shaped audio container — streamed, timeline-cued, scene-scoped — over the same
codecs as everything else.

## Editing implications

- **Replace streams in the `.big`**, keeping the index (`.csi`/`.idx`) pointing at them — the routing/payload
  discipline ([C19.2](../C19-Audio-Banks/02-snr-spt.md)): same-size in place, or re-stamp offsets on resize.
- **Decode with EA-XAS** — no new codec ([C20.4](../C20-Audio-Codecs/04-xas-ima.md)); match the rate
  ([C20.2](../C20-Audio-Codecs/02-replacement-rules.md)).
- **Keep cues aligned.** If you change a line's length, the schedule's timing ([C25.2](02-sequence-format.md))
  may need adjusting so it still lands with the action.
- **Preserve the index/event files.** They tie streams to the script; editing the `.big` without them
  desynchronises cues.

---

### Key takeaways

- Cutscene audio is the **NISAudio** set: `.big` (payload) + `.csi`(`MOIR`)/`.idx`/`.evt` (index/events) — a
  routing/payload container.
- The audio is the **shared codec layer** (EA-XAS/PCM) — decode it with the Chapter 20 decoder, nothing new.
- The event script **cues** streams by time, landing dialogue and sound with the action.
- NISAudio is separate because cutscene audio is streamed, timeline-cued, and scene-scoped.
- Edit streams in the `.big` with routing/payload discipline, decode with EA-XAS, and keep cues and index files
  aligned.

**Continue:** [C25.5 — Three-source playback assembly](05-playback-assembly.md) · [Chapter 25 hub](C25-NIS-Events.md)
