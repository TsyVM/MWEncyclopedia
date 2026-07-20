# C23.1 — The EA Multimedia Container

> **The one-sentence version:** a `.vp6` file opens with the `MVhd` EA-Multimedia header and muxes two streams
> — On2 VP6 video (`06PV`) and an EA-stream audio track (`SCHl`) — so you *demux* it, you don't decode a single
> raw stream.

[← Chapter 23 hub](C23-Video-VP6.md) · [Next: C23.2 — On2 VP6 video →](02-vp6-video.md)

---

## The header and the streams

The retail `blacklist_01_english_ntsc.vp6` begins:

```
+0x00  "MVhd"     EA Multimedia video header magic
+0x04  u32  0x20  header size
+0x08  "06PV"     video codec tag (VP60 / On2 VP6)
+0x0C  …          width, height, frame rate, frame count
…      "SCHl" "GSTR"   the audio stream's EA-stream blocks (Chapter 21)
```

So a `.vp6` is a **muxed container**: a header describing the movie, then **interleaved** video frames and
audio blocks. The `MVhd` magic identifies the EA Multimedia format; `06PV` names the video codec
([C23.2](02-vp6-video.md)); the `SCHl`/`GSTR` blocks are the audio track ([C23.3](03-audio-track.md)) — the
same EA-stream audio blocks as the music ([C21.2](../C21-Music-MUS-MPF/02-section-blocks.md)).

## Muxed, not concatenated

"Muxed" (multiplexed) means the video and audio are **interleaved** through the file, not stored as two
separate blocks — a chunk of video, then the audio for that time, then the next video, and so on. This is what
lets a player stream the movie: it reads a little ahead, decoding video and audio in lockstep, without loading
the whole file. Demuxing is separating the two interleaved streams back out ([C23.4](04-demux-transcode.md)).

```
[MVhd header][ V ][ A ][ V ][ A ][ V ][ A ] …     interleaved, streamable
```

## The extension names the container's codec

`.vp6` is slightly misleading: it names the *video codec*, but the file is a *container* carrying video **and**
audio. This is the same relationship as `.mp4` (a container, not a codec) or the game's own audio containers
([C20.1](../C20-Audio-Codecs/01-codec-set.md)) — the extension hints at the main content, but the file has
structure around it. Treating a `.vp6` as a raw VP6 stream (and feeding it straight to a VP6 decoder) fails,
because you'd feed it the `MVhd` header and the muxed audio too.

## Why EA used a container

Wrapping VP6 in the EA Multimedia container rather than shipping raw VP6 buys the essentials of any movie
format:

- **Synchronised audio.** Muxing interleaves audio with video so they stay in sync during streaming — a raw
  video stream has no audio.
- **Streamable.** The interleave lets the player read incrementally, decoding as it goes, rather than loading
  the whole movie.
- **Self-describing.** The `MVhd` header carries dimensions, frame rate, and stream layout, so the player knows
  how to present the movie without external metadata.

These are exactly what a container is *for*, and why the file is `MVhd`-wrapped VP6 rather than bare frames.

> ✅ *Verified:* `MVhd` magic, the `06PV` video tag, and `SCHl`/`GSTR` audio blocks are present in the retail
> `.vp6`, confirming a muxed EA Multimedia container.

## Editing implications

- **Demux before touching streams.** Separate video and audio first ([C23.4](04-demux-transcode.md)); don't
  treat the file as one stream.
- **Preserve the mux on rebuild.** A replacement movie must be re-muxed (interleaved) into the EA Multimedia
  container, not concatenated ([C23.6](06-replacing-movies.md)).
- **Keep the header honest.** Dimensions, frame rate, and counts in `MVhd` must match the actual streams.

---

### Key takeaways

- A `.vp6` is an **EA Multimedia container** (`MVhd`) muxing VP6 video (`06PV`) and an EA-stream audio track
  (`SCHl`).
- The streams are **interleaved** (muxed) for synchronised, streamable playback — not two concatenated blocks.
- The extension names the container's *video codec*; the file has structure (header + muxed audio) around it.
- The container gives synchronised audio, streamability, and self-description — why EA wrapped VP6.
- Demux before editing streams; re-mux on rebuild; keep the `MVhd` header consistent with the streams.

**Continue:** [C23.2 — On2 VP6 video](02-vp6-video.md) · [Chapter 23 hub](C23-Video-VP6.md)
