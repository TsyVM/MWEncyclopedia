# C23.3 — The EA-ADPCM Audio Track

> **The one-sentence version:** a movie's audio is an EA-stream track — `SCHl`/`GSTR` blocks carrying EA-ADPCM
> (XAS) or PCM — muxed alongside the video, which means cutscene audio decodes with the exact same codec layer
> as engine sound and music.

[← C23.2 — On2 VP6 video](02-vp6-video.md) · [Chapter 23 hub](C23-Video-VP6.md) ·
[Next: C23.4 — Demuxing & transcoding →](04-demux-transcode.md)

---

## The audio is EA-stream

Interleaved with the VP6 video ([C23.1](01-container.md)) is the movie's **audio track**, and it is not a new
format — it is the same **EA-stream** audio as the music: **`SCHl`/`GSTR`** blocks
([C21.2](../C21-Music-MUS-MPF/02-section-blocks.md)) carrying **EA-ADPCM XAS** or PCM samples. Verified: the
retail `.vp6` contains `SCHl`/`GSTR` blocks right after the video header. So the dialogue and effects in a
cutscene decode with the codec layer of [Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md) — specifically
EA-XAS, the engine/movie-audio codec ([C20.4](../C20-Audio-Codecs/04-xas-ima.md)).

## One codec layer, everywhere

This is the elegant payoff of the shared codec layer ([C20.1](../C20-Audio-Codecs/01-codec-set.md)): the audio
inside a *movie* is the same family as the audio inside a *bank* or a *music section*. So:

```
movie .vp6 → demux → audio track (SCHl/GSTR) → EA-XAS decode (C20.4) → PCM
                   → video track (VP6)        → VP6 decode (C23.2)   → frames
```

You already have the audio decoder — it's the one you built for banks and music. Decoding a cutscene's audio
adds nothing new beyond demuxing it out of the container. This is why the book teaches the codec layer once
([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)) and reuses it across banks, music, engine, and movies.

## The big-endian header, again

The `SCHl` audio header's format fields are **big-endian** ([C21.2](../C21-Music-MUS-MPF/02-section-blocks.md),
[C19.4](../C19-Audio-Banks/04-pt-records.md)) — the same convention as every EA audio-metadata header. So when
you read the movie audio track's rate and codec from its `SCHl`, byte-swap; reading them little-endian gives
nonsense, exactly as it does for bank `PT` and music sections. The trap is consistent across the whole audio
family, which is at least a mercy: learn it once.

> ✅ *Verified:* the retail `.vp6` carries `SCHl`/`GSTR` audio blocks — the EA-stream audio format shared with
> music ([C21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)); the audio codec is the EA-ADPCM (XAS) family.

## Sync with the video

The audio track's role is to stay **in sync** with the video during playback. The mux interleaves audio blocks
with the video frames they accompany ([C23.1](01-container.md)), and the player decodes both in lockstep,
presenting each frame with its matching audio. This is why the container muxes rather than concatenates: A/V
sync is a streaming property of the interleave, not something the player has to reconstruct.

For editing, sync is the constraint: a replacement audio track must have the **same duration** as the video (or
the video re-timed to match), or dialogue drifts against the picture ([C23.6](06-replacing-movies.md)).

## Editing implications

- **Decode movie audio with the EA-XAS decoder** you already have ([C20.4](../C20-Audio-Codecs/04-xas-ima.md)) —
  no new codec.
- **Read `SCHl` fields big-endian** ([C21.2](../C21-Music-MUS-MPF/02-section-blocks.md)).
- **Keep audio duration matched to video** on replacement, or A/V sync drifts ([C23.6](06-replacing-movies.md)).
- **Re-mux, don't concatenate** — a replaced audio track must be interleaved back with the video
  ([C23.4](04-demux-transcode.md)).

---

### Key takeaways

- Movie audio is an **EA-stream** track (`SCHl`/`GSTR`, EA-ADPCM XAS/PCM) muxed with the video.
- It's the **same codec layer** as banks/music (Chapter 20) — cutscene audio decodes with EA-XAS, nothing new.
- The `SCHl` format fields are **big-endian** — the consistent EA audio-header trap.
- The mux keeps A/V in **sync** during streaming; replacement audio must match the video's duration.
- Reuse the EA-XAS decoder, read headers big-endian, keep durations matched, and re-mux rather than
  concatenate.

**Continue:** [C23.4 — Demuxing & transcoding](04-demux-transcode.md) · [Chapter 23 hub](C23-Video-VP6.md)
