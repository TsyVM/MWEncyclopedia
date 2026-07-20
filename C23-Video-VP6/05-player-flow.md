# C23.5 — The Movie Player as a Flow State

> **The one-sentence version:** playing a movie is a flow state the game enters — it suspends gameplay, streams
> and decodes the container in sync, honours skip input, and on completion transitions to the next state — so
> the movie player is a *mode*, not just a file reader.

[← C23.4 — Demuxing & transcoding](04-demux-transcode.md) · [Chapter 23 hub](C23-Video-VP6.md) ·
[Next: C23.6 — Replacing a movie →](06-replacing-movies.md)

---

## A movie is a game state

The movie player is best understood not as a file format but as a **state the game enters**. When a cutscene
plays — the intro, a blacklist defeat, an attract loop — the game leaves interactive gameplay, enters the
movie-playback state, and returns to a new state when the movie ends. It is a scripted interlude, sequenced by
the game's flow ([this is the runtime frame/flow side of the engine](../README.md)):

```
gameplay / menu  ──play movie──▶  [ MOVIE PLAYBACK STATE ]  ──on end / skip──▶  next state
```

So a `.vp6` isn't played in a vacuum — something in the game's flow *enters* the movie state with a given file,
and the player *exits* to a designated next state. Understanding this is what lets you reason about *when* and
*why* a movie plays, not just *how* it decodes.

## What the state does

While in the movie-playback state, the player runs a tight loop:

1. **Stream** the container from disk incrementally (movies are large; they aren't fully loaded).
2. **Demux** the interleaved streams ([C23.1](01-container.md)).
3. **Decode** VP6 to frames and EA-XAS to PCM ([C23.2](02-vp6-video.md)–[C23.3](03-audio-track.md)).
4. **Present** each frame with its matching audio, in sync.
5. **Poll input** for a skip request.
6. **On completion or skip**, tear down and transition to the next state.

This is a self-contained mode: it owns the screen and audio, suspends the normal frame update, and hands
control back only when done.

## Skipping

Almost every movie is skippable, and skip is a first-class part of the state: pressing the skip button
**short-circuits** the playback loop and jumps straight to the exit transition. This is why the state must know
its *next state* up front — a skip needs to go somewhere immediately. For seeking to a clean stop, the player
can cut at a keyframe boundary ([C23.2](02-vp6-video.md)), but for a full skip it simply abandons decoding and
transitions.

> 🟡 *Reasoned:* the movie-playback loop and its enter/skip/exit transitions are described from the movie
> player's role as a game flow state; the ✅ verified facts are the container/codec structure
> ([C23.1](01-container.md)–[C23.3](03-audio-track.md)) the player consumes.

## Streaming, not loading

Because movies are large (tens of MB), the player **streams** rather than loads them — reading ahead just
enough to keep the decoders fed. This is the same residency logic as world streaming
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) applied to a linear medium: you never hold the
whole movie, you keep a sliding window resident. The mux ([C23.1](01-container.md)) is what makes this possible
— interleaved A/V lets the player read forward and decode both streams from one sequential read.

## Where movies fit in the flow

Movies punctuate the game's structure:

- **Attract mode** — the intro/demo loop that plays when the game idles at the title.
- **Story beats** — blacklist intros/defeats that bookend progression.
- **Menu backdrops** — animated backgrounds that loop behind the UI.

Each is a movie state entered by the flow at a defined point and exited to a defined next state — so a movie is
a node in the game's state machine, with the file as its content.

## Editing implications

- **Replacing a movie changes the content, not the flow.** The state still plays whatever `.vp6` is at the
  path; swap the file ([C23.6](06-replacing-movies.md)) and the same flow plays your movie.
- **Keep it skippable-friendly.** Ensure your movie has keyframes for clean cutting and isn't so long that the
  skip transition matters.
- **Match duration expectations** where a movie loops (menu backdrops) or bookends a beat — a wildly different
  length can feel wrong even if it plays.
- **Don't rely on changing the flow** from a file edit — which movie plays *where* is game logic, not the file.

---

### Key takeaways

- The movie player is a **flow state** the game enters: suspend gameplay → stream/demux/decode/present → skip or
  finish → next state.
- The playback loop streams (not loads), demuxes, decodes both streams in sync, polls skip, and transitions.
- **Skip** is first-class — it short-circuits to the exit, which is why the state knows its next state up front.
- Movies **stream** like the world does (a sliding window), enabled by the mux's interleave.
- Editing swaps the *content* (`.vp6`) but not the *flow*; keep movies keyframe-friendly and duration-appropriate.

**Continue:** [C23.6 — Replacing a movie](06-replacing-movies.md) · [Chapter 23 hub](C23-Video-VP6.md)
