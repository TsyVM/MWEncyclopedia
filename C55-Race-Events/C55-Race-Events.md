# Chapter 55 — Race Events & Game Modes

> **Goal of this chapter:** decode the event types that fill the career — the shared `RaceFlow` state machine
> (countdown → race → finish), the road races (Circuit, Sprint), the straight-line Drag minigame
> (`DragTachometer`), and the alternative modes (Speedtrap, Tollbooth, Lap Knockout) — plus the checkpoint system
> they share.

The career ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) is a ladder of *events*, and this
chapter decodes what those events *are*. Most Wanted's race modes — Circuit, Sprint, Drag, Speedtrap, Tollbooth,
Lap Knockout — are variations on a shared race framework, each with its own rules and scoring. This chapter
decodes the common `RaceFlow` machine they all run on and the distinct modes built on it, the events you play to
earn bounty ([C54.4](../C54-GameFlow-Blacklist/04-bounty-milestones.md)) and climb the Blacklist.

> **Verified against the executable.** The event types are named in `speed.exe`: **Circuit** (`CircuitMenu`),
> **Sprint** (`SprintMenu`), **Drag** (`DragMenu`, `DragTachometer`, `Dragster`), **Speedtrap** (`speedtrap`,
> `speedtrapjump`), **Tollbooth** (`TollboothMarker`, `TollboothStat`), **Lap Knockout** (`KnockoutRacer`), and
> checkpoint races (`checkpointrace`, `Checkpoint`/`CheckpointReached`). The race runs a **`RaceFlow`** machine —
> `RaceCountdown`, `RaceActivity`, `RaceCompleted`, `RaceFinished`, `RaceAbandoned`. Events are typed `EVENT_RIVAL`
> (Blacklist), `EVENT_ICON`, `EVENT_FENG`.

---

## Deep-dive pages

- [C55.1 — The RaceFlow machine](01-race-flow.md): the shared race lifecycle and checkpoints.
- [C55.2 — Circuit & Sprint](02-circuit-sprint.md): the two core road races.
- [C55.3 — Drag](03-drag.md): the straight-line shifting minigame.
- [C55.4 — Speedtrap, Tollbooth & Knockout](04-speed-modes.md): the alternative scoring modes.
- [C55.5 — Reading events in RE](05-reading-events.md): navigating the event system.

---

## 55.1 The RaceFlow machine

Every race runs a shared **`RaceFlow`** state machine ([C55.1](01-race-flow.md)): `RaceCountdown` (the 3-2-1 start)
→ `RaceActivity` (the race itself, hitting `Checkpoint`s) → `RaceCompleted`/`RaceFinished` (you finished) or
`RaceAbandoned` (you quit/failed). This lifecycle is common to *all* the modes — they differ in *rules* (what you're
doing) but share the *flow* (start, race, finish). The **checkpoint** system (`CheckpointReached`) is the shared
mechanism for tracking progress through a course.

## 55.2 Circuit & Sprint

The two core **road races** ([C55.2](02-circuit-sprint.md)) are **Circuit** (multi-lap around a closed loop —
`CircuitMenu`) and **Sprint** (a one-way dash from start to finish — `SprintMenu`). Both are wheel-to-wheel races
against AI racers ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md), the `AIGoalRacer` brain) through
checkpoints; they differ in *shape* — a lap you repeat vs. a point-to-point route. These are the bread-and-butter of
the career ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).

## 55.3 Drag

**Drag** ([C55.3](03-drag.md)) is a different game — a **straight-line** race where the skill is *shifting*, not
steering. The verified `DragTachometer` is its signature UI (the rev meter with the perfect-shift window), and
`EngineDragster`/`Dragster` ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) is its specialised
drivetrain. Steering is automated (you stay in your lane); you focus on the launch and the perfect shifts, dodging
traffic at extreme speed. It's a minigame within the game.

## 55.4 Speedtrap, Tollbooth & Knockout

Three **alternative-scoring** modes ([C55.4](04-speed-modes.md)) reward different skills: **Speedtrap** (`speedtrap`)
scores your *speed* through cameras along the route (fastest cumulative trap speed wins, not first place);
**Tollbooth** (`TollboothMarker`) is a *time trial* — reach each tollbooth before the clock expires, racing the
clock not rivals; **Lap Knockout** (`KnockoutRacer`) eliminates the *last-place* racer each lap until one remains.
Each reuses the `RaceFlow` machine ([C55.1](01-race-flow.md)) with different win conditions.

---

### Key takeaways

- All events run a shared **`RaceFlow`** machine — `RaceCountdown` → `RaceActivity` (through `Checkpoint`s) →
  `RaceFinished`/`RaceAbandoned` — differing in *rules*, sharing the *flow*.
- **Circuit** (multi-lap loop) and **Sprint** (point-to-point) are the core **road races** against AI racers.
- **Drag** is a **straight-line shifting** minigame — `DragTachometer` (perfect-shift meter), `EngineDragster`
  drivetrain, automated steering.
- **Speedtrap** (score by speed through traps), **Tollbooth** (beat the clock to each marker), and **Lap Knockout**
  (last-place eliminated each lap) are **alternative-scoring** modes.
- All modes **reuse the `RaceFlow` machine** with different win conditions — one framework, many games.

**Next:** [Chapter 56 — Customization: Performance & Visual](../C56-Customization/C56-Customization.md): the cars you
race with.
