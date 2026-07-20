# C54.1 — The GameFlow State Machine

> **The one-sentence version:** `GameFlowStates` is the game's top-level state machine — the phases the whole game
> moves between (front-end, track load, in-game, cutscene) — and it drives the resource-streaming phases, each
> state's transition loading the next state's resident set behind a loading screen.

[← Chapter 54 hub](C54-GameFlow-Blacklist.md) · [Next: C54.2 — The CareerManager →](02-career-manager.md)

---

## The top-level machine

Above every individual system is the **`GameFlowStates`** machine — the *master state machine* that governs what
the game is *doing* at the largest scale. It's the same `GameFlow` the streaming manager keys off
([Chapter 38](../C38-Resource-Streaming-Residency/04-gameflow.md)), and its states are the game's top-level phases:

- **Front-end** (`GameFlowLoadingFrontEnd`, `GameFlowLoadingFrontEndPart`) — the menus, garage, Blacklist screen
  ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)).
- **Track load** (`GameFlowLoadTrack`) — loading a race or the open world
  ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).
- **In-game** — actually playing (racing, free-roam, pursuit).
- **Cutscene** — the NIS/cinematic sequences ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)).

`GameFlowStates` moves between these, and the transition drives everything else — the loading, the camera handover
([C53.1](../C53-Cameras-Director/01-two-systems.md)), the music ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)).
It's the conductor of the whole game's structure.

> ✅ *Verified:* `GameFlowStates`, `GameFlowLoadTrack`, `GameFlowLoadingFrontEnd`, and `GameFlowLoadingFrontEndPart`
> are present in `speed.exe` — the top-level state machine and its transition states.

## States drive resource residency

The crucial role of `GameFlowStates` is that **each state defines a resource residency** — this is the link to the
streaming manager ([Chapter 38](../C38-Resource-Streaming-Residency/04-gameflow.md)):

- **Each state has a manifest** ([C38.5](../C38-Resource-Streaming-Residency/05-manifests.md)) — the front-end
  loads the front-end resources; in-game loads the world and cars.
- **A transition loads the next state's set** — entering `GameFlowLoadTrack` acquires the track's resources
  ([C38.3](../C38-Resource-Streaming-Residency/03-refcounting.md)), blocking on the essentials
  ([C38.6](../C38-Resource-Streaming-Residency/06-blocking-budgets.md)) behind a loading screen.
- **The transition tears down the old set** — leaving the front-end releases its resources, freeing memory for the
  track.

So the `GameFlowStates` machine is *why* loading screens happen where they do
([C38.6](../C38-Resource-Streaming-Residency/06-blocking-budgets.md)) — each state change is a resource swap
(release the old, load the new). The `GameFlowLoadingFrontEnd`/`GameFlowLoadTrack` transition *states* are literally
the loading phases: the game is in "loading the front-end" or "loading the track" state while the streamer works.
The state machine and the streaming manager are two views of one thing — the game's phase, and its resident memory.

## Front-end ↔ in-game: the core loop

The most-traversed transition is **front-end ↔ in-game** — the loop between menus and play:

```
FrontEnd (menus, garage, Blacklist)
   → GameFlowLoadTrack (load the event/world) ← loading screen
      → In-Game (race / free-roam / pursuit)
         → (event ends) → GameFlowLoadingFrontEnd (back to menus) ← loading screen
            → FrontEnd ...
```

You spend the game bouncing between the front-end (choosing an event, tuning your car,
[Chapter 56](../C56-Customization/C56-Customization.md)) and the track (playing it). Each crossing is a
`GameFlowStates` transition with its resource swap ([above](#states-drive-resource-residency)). This loop is the
*rhythm* of the game — pick, play, return — and `GameFlowStates` is its engine. The career
([C54.2](02-career-manager.md)) rides on top: it's the front-end deciding *which* event to load next.

## Why a state machine

Structuring the game's top level as an explicit state machine (rather than ad-hoc mode flags) is clean, robust
design:

- **One authority for "what's happening."** `GameFlowStates` is the single source of truth for the game's phase —
  every system can ask it and react (the camera, [Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md); the
  music, [Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md); the streamer,
  [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
- **Transitions are explicit.** Moving between phases is a defined transition with defined work (load this, release
  that) — no tangled implicit mode changes.
- **Resource residency is bounded per state** — each state's memory is its manifest
  ([C38.5](../C38-Resource-Streaming-Residency/05-manifests.md)), so memory is predictable and phase-scoped.

So `GameFlowStates` is the *backbone* of the whole game's structure — the master phase machine that every other
system hangs off. Understanding it is understanding how Most Wanted is *organised* at the top: a set of states, with
transitions that swap resources and hand off control, driving the loop from menu to race and back.

## RE implications

- **`GameFlowStates`** is the game's top-level state machine — front-end, track load, in-game, cutscene.
- **Each state defines a resource residency** — transitions load/release manifests
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)); the loading states *are*
  the loading screens.
- **Front-end ↔ in-game** is the core loop — pick an event, load, play, return.
- **A state machine** gives one phase authority, explicit transitions, and bounded per-state memory.

---

### Key takeaways

- **`GameFlowStates`** is the game's **top-level state machine** — the phases (front-end, `GameFlowLoadTrack`,
  in-game, cutscene) the whole game moves between.
- **Each state defines a resource residency** — transitions load the next state's manifest and release the old
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)); the `GameFlowLoading*`
  states **are** the loading screens.
- The **front-end ↔ in-game** loop (pick → load → play → return) is the game's rhythm, each crossing a
  `GameFlowStates` transition with a resource swap.
- A **state machine** gives one authority for the game's phase, explicit transitions, and bounded per-state memory —
  the backbone every system (camera, music, streamer) hangs off.
- The career ([C54.2](02-career-manager.md)) **rides on top** — it decides which event the front-end loads next.

**Continue:** [C54.2 — The CareerManager](02-career-manager.md) · [Chapter 54 hub](C54-GameFlow-Blacklist.md)
