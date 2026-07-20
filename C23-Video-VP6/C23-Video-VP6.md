# Chapter 23 — Video (EA Multimedia / VP6)

> **Goal of this chapter:** decode the game's movies — the EA Multimedia container holding On2 VP6 video and
> EA-ADPCM audio — well enough to demux, play, transcode, and replace them, and to understand the movie player
> as a flow state the game enters and leaves.

The FMVs, the attract-mode intro, the blacklist cutscenes, and the animated menu backdrops are **video files** —
`.vp6` — and they are not raw codec streams but **EA Multimedia containers** wrapping On2 VP6 video and an
EA-ADPCM audio track. The happy news: this is a well-known industry container/codec pair, so mainstream
multimedia stacks already handle it. This chapter decodes the container and explains how the game plays it.

> **Verified against retail data.** `MOVIES/blacklist_01_english_ntsc.vp6` opens with the EA Multimedia magic
> **`MVhd`**, immediately followed by the video codec tag **`06PV`** (`VP60`/On2 VP6) and, further in, an
> **`SCHl`/`GSTR`** audio section — the same EA-stream audio blocks as the music
> ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)). So a `.vp6` is a muxed container: VP6 video + an
> EA-stream audio track.

---

## Deep-dive pages

- [C23.1 — The EA Multimedia container](01-container.md): the `MVhd` header, the muxed streams, and why `.vp6`
  names the container not a raw codec.
- [C23.2 — On2 VP6 video](02-vp6-video.md): the video codec, why EA used a licensed general-purpose codec, and
  how frames are carried.
- [C23.3 — The EA-ADPCM audio track](03-audio-track.md): the `SCHl`/`GSTR` audio muxed alongside the video.
- [C23.4 — Demuxing & transcoding](04-demux-transcode.md): using a standard multimedia library rather than
  reinventing the decoder.
- [C23.5 — The movie player as a flow state](05-player-flow.md): entering, playing, skipping, and leaving a
  movie.
- [C23.6 — Replacing a movie](06-replacing-movies.md): re-encoding to VP6-in-EA and fitting the game's
  expectations.

---

## 23.1 `.vp6` is a container, not a codec

The extension names the *container's main codec*, but the file is a **container**: a header plus **muxed
streams** — VP6 video interleaved with an EA-stream audio track ([C23.1](01-container.md)). The header magic is
**`MVhd`** (EA Multimedia video header); the video codec tag `06PV` follows; the audio is carried in `SCHl`
sections. So opening a `.vp6` means *demuxing* — separating the video and audio streams — not decoding a single
raw stream ([C23.4](04-demux-transcode.md)).

## 23.2 On2 VP6 video

The video codec is **On2 VP6** (`vp6f`), a licensed general-purpose video codec of the era (the same VP6 used
in Flash video). EA chose it rather than a bespoke codec because a proven general-purpose codec with a known
container is far cheaper than inventing one and already hit the quality/size trade the FMVs needed
([C23.2](02-vp6-video.md)). The frames are standard VP6, so any VP6 decoder produces the pictures.

## 23.3 The audio track

The audio muxed alongside the video is an **EA-stream** track — `SCHl`/`GSTR` blocks
([C21.2](../C21-Music-MUS-MPF/02-section-blocks.md)) carrying **EA-ADPCM (XAS)** or PCM, the *same* audio
family as engine sound and music ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)). So the movie's audio
reuses the codec layer you already know — decoding a cutscene's dialogue is the same EA-XAS decode as a car
engine ([C23.3](03-audio-track.md)).

## 23.4 Don't reinvent it — demux with a standard stack

Because VP6-in-EA is a **well-known industry format**, mainstream multimedia libraries (the `ea` demuxer plus a
`vp6f` decoder in FFmpeg-family stacks) already read it. The right approach is to **demux and decode with a
standard library**, not to hand-roll a VP6 decoder — video is solved, and reinventing it is pointless
([C23.4](04-demux-transcode.md)). This is the one place in the book where the correct answer is "use the
existing tool."

## 23.5 The player is a flow state

At runtime, playing a movie is a **flow state** the game enters: it suspends normal gameplay, streams and
decodes the container, presents video + audio, honours skip input, and on completion (or skip) transitions to
the next game state ([C23.5](05-player-flow.md)). The movie player is less a file format than a *mode* — a
scripted interlude between interactive states.

---

### Key takeaways

- `.vp6` files are **EA Multimedia containers** (`MVhd`), not raw codecs — VP6 video muxed with an EA-stream
  audio track.
- Video is **On2 VP6** (`06PV`/`vp6f`), a licensed general-purpose codec; audio is **EA-ADPCM XAS**/PCM in
  `SCHl` blocks.
- The audio reuses the shared codec layer (Chapter 20) — cutscene audio decodes like engine/music audio.
- **Demux and decode with a standard multimedia stack** — VP6-in-EA is well-known; don't reinvent it.
- The movie player is a **flow state** the game enters, plays, allows skipping, and leaves.

**Next:** [Chapter 24 — Animations & Cutscenes: the NIS Object](../C24-NIS-Animation/C24-NIS-Animation.md): the
*rendered* (non-video) cutscenes and their animation format.
