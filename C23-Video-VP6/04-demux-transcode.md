# C23.4 — Demuxing & Transcoding

> **The one-sentence version:** because VP6-in-EA is a well-known industry format, the right approach is to
> demux and decode with a standard multimedia library (the `ea` demuxer + `vp6f` decoder) rather than
> hand-rolling a decoder — this is the one place in the book where "use the existing tool" is the answer.

[← C23.3 — The EA-ADPCM audio track](03-audio-track.md) · [Chapter 23 hub](C23-Video-VP6.md) ·
[Next: C23.5 — The movie player as a flow state →](05-player-flow.md)

---

## Use a standard multimedia stack

Almost everything in this book is reverse-engineered because the formats are EA-custom. Video is the exception:
the EA Multimedia container and On2 VP6 are **published, well-supported** formats that mainstream multimedia
libraries already handle. FFmpeg-family stacks carry an **`ea` demuxer** (for the EA Multimedia container) and
a **`vp6f` decoder** (for the video), plus `adpcm_ea_xas` for the audio ([C23.3](03-audio-track.md)). So:

```bash
# play / inspect
ffprobe blacklist_01_english_ntsc.vp6      # shows the ea container, vp6f video, EA-XAS audio
ffplay  blacklist_01_english_ntsc.vp6      # plays it directly

# transcode out (to edit in standard tools)
ffmpeg -i movie.vp6 -c:v png frames_%05d.png -c:a pcm_s16le audio.wav
```

The tools demux the interleaved streams ([C23.1](01-container.md)), decode VP6 to frames and EA-XAS to PCM, and
hand you standard media. **Do not hand-roll a VP6 decoder** — it's a large, solved problem, and the existing
stacks are correct and fast.

## Demux, then decode

Under the hood the pipeline is:

```
.vp6 (EA Multimedia container)
  → ea demuxer: split into VP6 video packets + EA-XAS audio packets
  → vp6f decoder:      VP6 packets  → frames (RGB/YUV)
  → adpcm_ea_xas:      audio packets → PCM
  → present in sync (C23.5) / or write to files (transcode)
```

Demuxing is the container work ([C23.1](01-container.md)); decoding is off-the-shelf for both streams. This
cleanly separates the two concerns: the container (EA-specific, but standard-supported) and the codecs (VP6 and
EA-XAS, both handled by the library).

## Transcoding out and in

For editing, you transcode **out** to standard formats, edit there, and transcode **in** to VP6-in-EA:

- **Out** — `ffmpeg -i movie.vp6 …` to PNG frames + WAV (or an MP4 intermediate). Now you can edit the video
  and audio in ordinary tools.
- **In** — re-encode your edited video to VP6 and mux into the EA Multimedia container with the EA-XAS audio
  ([C23.6](06-replacing-movies.md)). This is the harder direction, because you must produce a container the
  game's player accepts (right dimensions, frame rate, mux layout).

The out direction is trivial (standard tools); the in direction is where care is needed
([C23.6](06-replacing-movies.md)).

> ✅ *Verified:* `.vp6` files are EA Multimedia containers (`ea` demuxer) carrying `vp6f` video and EA-XAS/PCM
> audio — decodable by standard multimedia stacks without reverse-engineering the codec.

## Why this is the right call

Reinventing the video decoder would be the classic reverse-engineering trap — enormous effort to reproduce
something already available and correct:

- **Correctness for free.** A mature VP6 decoder handles every frame type and edge case; a hand-rolled one
  wouldn't, and video decode bugs are subtle.
- **Speed.** Optimised library decoders are far faster than a from-scratch implementation.
- **Focus.** The book's reverse-engineering effort belongs on the EA-custom formats (geometry, audio banks,
  animation), not on a licensed commodity codec.

So the discipline here is knowing *when not to reverse-engineer* — video is solved, and the leverage is in
using the solution.

## Editing implications

- **Transcode out with standard tools** — trivial and lossless-ish to an intermediate.
- **Edit in ordinary video tools**, then re-encode to VP6.
- **Mux back into EA Multimedia** carefully ([C23.6](06-replacing-movies.md)) — the in direction is the real
  work.
- **Validate with `ffprobe`** — confirm your rebuilt file demuxes as `ea`/`vp6f`/EA-XAS before shipping it.

---

### Key takeaways

- VP6-in-EA is a **standard-supported** format — demux with the `ea` demuxer, decode with `vp6f` + `adpcm_ea_xas`.
- **Don't hand-roll a VP6 decoder** — it's solved; use FFmpeg-family stacks.
- Pipeline: demux (container) → decode both streams (off-the-shelf) → present or transcode.
- Transcoding out is trivial; transcoding *in* (re-encode VP6 + re-mux) is the careful direction.
- Know when *not* to reverse-engineer — video is a commodity codec; spend RE effort on EA-custom formats.

**Continue:** [C23.5 — The movie player as a flow state](05-player-flow.md) · [Chapter 23 hub](C23-Video-VP6.md)
