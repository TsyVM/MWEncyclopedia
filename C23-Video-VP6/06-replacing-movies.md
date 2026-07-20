# C23.6 — Replacing a Movie

> **The one-sentence version:** to replace a movie, edit in standard tools, then re-encode to On2 VP6 and mux
> back into the EA Multimedia container with an EA-XAS audio track that matches the video's duration and the
> game's expected dimensions/frame rate.

[← C23.5 — The movie player as a flow state](05-player-flow.md) · [Chapter 23 hub](C23-Video-VP6.md) ·
[Next: Chapter 24 — Animations & Cutscenes: the NIS Object →](../C24-NIS-Animation/C24-NIS-Animation.md)

---

## Out is easy, in is careful

Movie replacement is asymmetric ([C23.4](04-demux-transcode.md)):

- **Out** — transcode the original to standard formats (PNG frames + WAV, or an MP4) with off-the-shelf tools;
  edit freely.
- **In** — re-encode your edited video to **VP6** and **mux** it into the **EA Multimedia container** with an
  **EA-XAS** (or PCM) audio track. This is the direction that requires care, because you must produce a
  container the game's player accepts.

```bash
# out (edit anywhere)
ffmpeg -i original.vp6 edited_intermediate.mp4

# in (produce a game-compatible movie)
ffmpeg -i my_edit.mp4 -c:v vp6f -c:a adpcm_ea_xas -f ea my_movie.vp6
```

## Match the game's expectations

The player was authored around the original movie's parameters, so a replacement should match them unless you
know the player tolerates otherwise:

- **Dimensions.** Keep the original width/height ([C23.1](01-container.md)); a different resolution may display
  wrong or not at all.
- **Frame rate.** Match it, so timing and any expected duration hold.
- **Container/codec tags.** Produce a genuine EA Multimedia (`ea`) container with `vp6f` video and EA-XAS/PCM
  audio — the tags the demuxer and player expect ([C23.4](04-demux-transcode.md)).
- **Audio format.** EA-XAS at the original rate, in sync ([C23.3](03-audio-track.md)).

The safest replacement keeps the *envelope* identical (dimensions, frame rate, codecs, roughly the duration)
and changes only the *content*.

## A/V sync and duration

The one thing that ruins a replaced movie even when it plays is **drift**: audio out of sync with video. Guard
it by:

- **Matching audio and video duration** — the audio track must be as long as the video (or the video re-timed).
- **Muxing, not concatenating** — the streams must be interleaved so the player presents each frame with its
  audio ([C23.1](01-container.md)); a standard muxer (`-f ea`) does this.
- **Testing the whole movie** — drift often appears only minutes in, so watch to the end.

Duration also matters for **looping** movies (menu backdrops) and **bookending** ones (story beats) — a
replacement of a very different length can feel wrong even if technically valid ([C23.5](05-player-flow.md)).

## Keyframes for skipping

Ensure your re-encode has **keyframes** at reasonable intervals ([C23.2](02-vp6-video.md)) so the player can cut
cleanly on skip and any seeking lands well. A VP6 encode with a sane keyframe interval (e.g. every couple of
seconds) is the norm; an encode with only one keyframe at the start makes seeking coarse.

## Verify before shipping

Validate the rebuilt movie the way the game will read it:

1. **`ffprobe` it** — confirm it demuxes as `ea` container, `vp6f` video, EA-XAS/PCM audio, with the right
   dimensions and frame rate.
2. **Play it** end-to-end — video and audio in sync throughout, no drift.
3. **Check duration** against the original if it loops or bookends.
4. **Test in game** — drop it at the movie's path and confirm the flow state ([C23.5](05-player-flow.md)) plays
   it, and that skip works.

The `ffprobe` check is the container equivalent of re-parsing an edited file — it confirms the game's demuxer
will accept your rebuild before you test in-engine.

## Editing implications, summarised

- **Transcode out, edit, re-encode to VP6, mux into EA** — the standard tools do all four.
- **Match dimensions, frame rate, codecs, and duration** to the original's envelope.
- **Keep A/V in sync** (matched duration, real mux); **keep keyframes** for skipping.
- **Verify with `ffprobe`, a full playback, and an in-game test** before shipping.

---

### Key takeaways

- Replacement is asymmetric: transcoding **out** is trivial; re-encoding **in** (VP6 + EA mux) is the careful
  part.
- Match the game's envelope — dimensions, frame rate, `ea`/`vp6f`/EA-XAS codecs — and change only the content.
- Guard **A/V sync**: matched audio/video duration, real muxing (not concatenation), and full-length testing.
- Include **keyframes** so the player skips and seeks cleanly.
- Verify with `ffprobe`, an end-to-end play, a duration check, and an in-game test.

**Continue:** [Chapter 24 — Animations & Cutscenes: the NIS Object](../C24-NIS-Animation/C24-NIS-Animation.md) ·
[Chapter 23 hub](C23-Video-VP6.md)
