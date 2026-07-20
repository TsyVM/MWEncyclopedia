# C23.2 — On2 VP6 Video

> **The one-sentence version:** the video codec is On2 VP6 (`vp6f`) — a licensed, general-purpose codec of the
> era — carried as standard VP6 frames in the container, so any VP6 decoder produces the pictures and EA never
> had to write a video codec.

[← C23.1 — The EA Multimedia container](01-container.md) · [Chapter 23 hub](C23-Video-VP6.md) ·
[Next: C23.3 — The EA-ADPCM audio track →](03-audio-track.md)

---

## On2 VP6

The `06PV` tag ([C23.1](01-container.md)) is `VP60` — **On2 VP6**, a video codec licensed from On2
Technologies (the lineage that later became VP8/VP9/WebM). VP6 was ubiquitous in the mid-2000s — it was the
codec behind Flash video — so it was a mature, well-supported choice. The frames in the container are **standard
VP6**, meaning any VP6 decoder decodes them; EA's contribution is the *container* ([C23.1](01-container.md)),
not the video codec.

## Why a licensed general-purpose codec

EA had bespoke formats for geometry, audio, and animation — why license a third-party video codec instead of
rolling their own?

- **Video is solved, and hard.** A competitive video codec is an enormous engineering effort (motion
  compensation, transform coding, entropy coding, rate control). Licensing a proven one (VP6) was far cheaper
  than matching it in-house.
- **The trade was already made.** VP6 hit the quality-per-bitrate the FMVs needed at the disc/streaming budgets
  of the era — a solved trade-off EA could simply buy.
- **Tooling existed.** Encoders, decoders, and pipelines for VP6 were mature, so EA's content team could author
  movies with off-the-shelf tools.

This is the pragmatic inverse of the rest of the engine: where geometry and audio are custom (tuned to the
game's needs), video is a commodity EA sensibly licensed rather than reinvented.

## Standard frames, standard decode

Because the frames are standard VP6, decoding is a solved problem:

```
demux (C23.4) → VP6 frame stream → any VP6 decoder → RGB/YUV frames → present
```

The decoder handles VP6's internals (keyframes and inter-frames, motion vectors, the transform); you don't
reimplement them. This is the crucial practical point of the whole chapter: **the pictures come from a standard
decoder**, so movie work is demuxing + off-the-shelf decoding, not codec reverse-engineering
([C23.4](04-demux-transcode.md)).

> ✅ *Verified:* the video codec tag is `06PV` (`VP60`/On2 VP6); the archive confirms `.vp6` files as EA
> Multimedia containers carrying `vp6f` video decoded by standard stacks.
> 🟡 *Reasoned:* VP6's internal frame coding is the published On2 VP6 spec (out of scope to re-derive here); the
> container's use of it is verified.

## Keyframes and inter-frames

Like any modern video codec, VP6 mixes **keyframes** (self-contained pictures) and **inter-frames** (encoded as
differences from previous frames). This matters for two practical reasons:

- **Seeking/skipping** ([C23.5](05-player-flow.md)) lands cleanly only on keyframes, so a player skips to the
  next keyframe.
- **Editing** ([C23.6](06-replacing-movies.md)) can't just replace arbitrary frames — inter-frames depend on
  their references, so you re-encode whole segments, not individual frames.

The decoder tracks the reference frames internally; you only care about this structure when seeking or
re-encoding.

## Editing implications

- **Decode with a standard VP6 decoder** — don't reimplement VP6 ([C23.4](04-demux-transcode.md)).
- **Re-encode to VP6** for replacement — encode your video to VP6 and re-mux
  ([C23.6](06-replacing-movies.md)); you can't hand-edit VP6 frames.
- **Respect keyframe structure** for seeking and cutting — operate on keyframe boundaries.
- **Match dimensions/frame rate** to the container header ([C23.1](01-container.md)) so the player presents it
  correctly.

---

### Key takeaways

- The video codec is **On2 VP6** (`06PV`/`vp6f`) — a licensed, general-purpose codec; the frames are standard
  VP6.
- EA licensed it because video is solved and hard: cheaper and better than a bespoke codec, with mature tooling.
- Decoding is off-the-shelf — demux, then hand frames to any VP6 decoder; no codec reverse-engineering needed.
- VP6 mixes keyframes and inter-frames; this governs seeking (land on keyframes) and editing (re-encode
  segments).
- Decode and re-encode with standard VP6 tools; respect keyframes; match the container's dimensions/frame rate.

**Continue:** [C23.3 — The EA-ADPCM audio track](03-audio-track.md) · [Chapter 23 hub](C23-Video-VP6.md)
