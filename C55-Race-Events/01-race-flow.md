# C55.1 — The RaceFlow Machine

> **The one-sentence version:** every race runs a shared `RaceFlow` state machine — `RaceCountdown` (3-2-1) →
> `RaceActivity` (racing through checkpoints) → `RaceCompleted`/`RaceFinished` or `RaceAbandoned` — common to all
> modes, which differ in rules but share this lifecycle.

[← Chapter 55 hub](C55-Race-Events.md) · [Next: C55.2 — Circuit & Sprint →](02-circuit-sprint.md)

---

## The shared lifecycle

Under every event type ([Chapter 55](C55-Race-Events.md)) is one shared **`RaceFlow`** state machine — the
lifecycle every race runs, regardless of its rules:

```
RaceCountdown   — the 3-2-1 start; cars staged, engines revving
   → RaceActivity   — the race is live; drive the course, hit checkpoints
      → RaceFinished / RaceCompleted   — you completed it (won or placed)
      → RaceAbandoned                   — you quit, failed, or DNF'd
```

So a Circuit, a Sprint, a Drag, a Speedtrap all *start* the same (countdown), *run* the same (activity through
checkpoints), and *end* the same (finished or abandoned) — the *flow* is shared. What differs is the *rules* inside
`RaceActivity` (what counts as winning, [C55.2](02-circuit-sprint.md)–[C55.4](04-speed-modes.md)). This is the same
pattern as the mechanics ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) — one framework, many
configurations.

> ✅ *Verified:* `RaceCountdown`, `RaceActivity`, `RaceCompleted`, `RaceFinished`, and `RaceAbandoned` are present
> in `speed.exe` — the race lifecycle states; `RaceFlow` is the machine. `Checkpoint`/`CheckpointReached` are the
> progress mechanism.

## Checkpoints: tracking progress

The shared mechanism for *tracking progress* through a race is the **checkpoint** system (`Checkpoint`,
`CheckpointReached`, `CheckpointPos`, `checkpointrace`):

- **A course is a sequence of checkpoints** — waypoints along the route the racer must pass, in order.
- **`CheckpointReached`** fires as you pass each — advancing your progress, updating your position vs. rivals.
- **The finish is the last checkpoint** — reaching it completes the race (`RaceFinished`).

So checkpoints are how the game *knows where you are* in a race — your progress is "which checkpoint you've reached."
This drives the positioning (who's ahead), the lap counting (Circuit, [C55.2](02-circuit-sprint.md)), the timing
(Tollbooth, [C55.4](04-speed-modes.md)), and the minimap ([Chapter 29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md))
route display. Checkpoints are the *skeleton* of a course — the ordered waypoints that define the route and measure
progress along it, shared by every mode.

## Countdown and staging

The race begins with **`RaceCountdown`** — the staged 3-2-1 start:

- **Cars are placed** at the start line in grid positions (you and the AI racers,
  [Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)).
- **The countdown runs** — 3, 2, 1, GO — a fixed pre-race beat (with its camera,
  [Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md), and audio).
- **Control releases** on GO — `RaceActivity` begins, and everyone launches (a good launch,
  [C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md), matters).

So the countdown is the *staging* phase — a controlled moment before the chaos, setting the grid and building
tension. It's a distinct state (`RaceCountdown`) because during it the cars are *held* (no driving yet) — the race
proper (`RaceActivity`) only begins on GO. This clean separation (stage, then race) is why the start is always
orderly, and why a jumped start or a perfect launch is a discrete, readable moment.

## Finish and abandon

A race ends in one of two states:

- **`RaceFinished`/`RaceCompleted`** — you reached the final checkpoint. Your result (position, time, or score
  depending on mode, [C55.4](04-speed-modes.md)) is recorded, rewards granted
  ([C54.4](../C54-GameFlow-Blacklist/04-bounty-milestones.md)), and control returns to the front-end via
  `GameFlowStates` ([C54.1](../C54-GameFlow-Blacklist/01-gameflow-states.md)) — often via a finish camera
  (`CameraPhotoFinish`, [C53.4](../C53-Cameras-Director/04-camera-moments.md)).
- **`RaceAbandoned`** — you quit, were disqualified, or failed the mode's condition (ran out of time in Tollbooth,
  eliminated in Knockout, [C55.4](04-speed-modes.md)). No reward; back to the front-end.

So `RaceFlow` always resolves to *finished* or *abandoned*, and either way returns control to the career
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)). This clean resolution is why a race always ends
somewhere definite — you either complete it (with a result) or abandon it (without) — and the career picks up from
there. The two-outcome machine keeps the game's flow ([C54.1](../C54-GameFlow-Blacklist/01-gameflow-states.md))
tidy: race → result → back to the career loop.

## Why a shared flow

Building all modes on one `RaceFlow` machine (rather than separate per-mode code) is the engine's composition
economy ([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)):

- **One lifecycle, many modes.** Countdown, activity, finish/abandon are written once; each mode plugs its *rules*
  into `RaceActivity` ([C55.2](02-circuit-sprint.md)–[C55.4](04-speed-modes.md)). A new mode is new *rules*, not a
  new flow.
- **Shared infrastructure.** Checkpoints, the countdown, the finish handling, the reward hookup
  ([C54.4](../C54-GameFlow-Blacklist/04-bounty-milestones.md)) are common — every mode gets them for free.
- **Consistent feel.** Every race *starts* and *ends* the same way, so the modes feel like variations of one game,
  not disparate minigames.

So `RaceFlow` is the *chassis* of the event system — the shared lifecycle onto which each mode bolts its rules. This
is why Most Wanted has so many event types ([Chapter 55](C55-Race-Events.md)) without proportional complexity: they
share the flow, and differ only in the win condition. Understanding `RaceFlow` is understanding the common structure
beneath every race.

## RE implications

- **`RaceFlow`** is the shared race machine — `RaceCountdown` → `RaceActivity` → `RaceFinished`/`RaceAbandoned`.
- **Checkpoints** track progress — the ordered waypoints every mode uses ([C55.2](02-circuit-sprint.md)).
- **Countdown stages** the grid; **finish/abandon** resolves cleanly to the career
  ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).
- **One shared flow, many modes** — modes plug rules into `RaceActivity`; the lifecycle is common.

---

### Key takeaways

- Every event runs the shared **`RaceFlow`** machine — **`RaceCountdown`** (3-2-1) → **`RaceActivity`** (race
  through checkpoints) → **`RaceFinished`/`RaceCompleted`** or **`RaceAbandoned`**.
- **Checkpoints** (`CheckpointReached`) are the shared progress mechanism — ordered waypoints that define the route
  and measure position, laps, and timing.
- The **countdown stages** the grid (cars held until GO); the **finish/abandon** resolves cleanly back to the career
  loop ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).
- All modes **plug their rules into `RaceActivity`** — the lifecycle (start, race, finish) is common; only the *win
  condition* differs.
- `RaceFlow` is the **chassis** of the event system — many modes, one shared flow, minimal added complexity.

**Continue:** [C55.2 — Circuit & Sprint](02-circuit-sprint.md) · [Chapter 55 hub](C55-Race-Events.md)
