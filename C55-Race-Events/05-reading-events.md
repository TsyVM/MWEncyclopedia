# C55.5 — Reading Events in RE

> **The one-sentence version:** navigate the events by the `RaceFlow` states, the mode strings (`CircuitMenu`/
> `SprintMenu`/`DragMenu`/`speedtrap`/`TollboothMarker`/`KnockoutRacer`), the checkpoint system, and the
> `EVENT_*` types — reading the event roster as one shared flow with per-mode win conditions.

[← C55.4 — Speedtrap, Tollbooth & Knockout](04-speed-modes.md) · [Chapter 55 hub](C55-Race-Events.md) ·
[Next: Chapter 56 — Customization: Performance & Visual →](../C56-Customization/C56-Customization.md)

---

## Anchors for event RE

The event system is anchored on verified strings:

- **The `RaceFlow` states** — `RaceCountdown`, `RaceActivity`, `RaceCompleted`, `RaceFinished`, `RaceAbandoned`
  ([C55.1](01-race-flow.md)).
- **The mode strings** — `CircuitMenu`, `SprintMenu`, `DragMenu`, `speedtrap`, `TollboothMarker`, `KnockoutRacer`
  ([C55.2](02-circuit-sprint.md)–[C55.4](04-speed-modes.md)).
- **The checkpoint system** — `Checkpoint`, `CheckpointReached`, `checkpointrace` ([C55.1](01-race-flow.md)).
- **The event types** — `EVENT_RIVAL`, `EVENT_ICON`, `EVENT_FENG`.

From these, the event roster is navigable: the shared flow, the modes, the checkpoints, and the event typing.

## The RE workflow

Reading the events:

1. **Trace `RaceFlow`** — the shared lifecycle ([C55.1](01-race-flow.md)); every mode runs it.
2. **Identify the modes** — the mode strings ([C55.2](02-circuit-sprint.md)–[C55.4](04-speed-modes.md)); each is a
   win condition on `RaceFlow`.
3. **Map the checkpoints** — the course structure ([C55.1](01-race-flow.md)) every mode uses.
4. **Follow the event types** — `EVENT_RIVAL` (Blacklist, [Chapter 54](../C54-GameFlow-Blacklist/03-the-blacklist.md))
   vs. regular events.

The output is the full event picture: the flow, the modes, the courses, and the typing.

## One flow, six modes: the pattern

The organising insight ([C55.1](01-race-flow.md)) is that the whole event roster is **one flow, many win
conditions**:

```
RaceFlow (shared): countdown → activity (checkpoints) → finish/abandon
   + Circuit     → win by position, multi-lap loop
   + Sprint      → win by position, point-to-point
   + Drag        → win by position, straight-line + shifting
   + Speedtrap   → win by cumulative speed
   + Tollbooth   → win by beating the clock
   + Knockout    → win by surviving elimination
```

Six modes, one shared machine, each differing only in the *win condition* plugged into `RaceActivity`. This is the
data-over-code / composition pattern ([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)) that recurs
throughout the engine — a shared framework configured into variants. Recognising it makes the event system simple:
learn `RaceFlow` once, and each mode is *just its win condition*. The engine gets six event types for barely more
than the cost of one.

## Events connect to the career

The events are where the career ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) meets the
simulation ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)):

- **`EVENT_RIVAL`** ([above](#anchors-for-event-re)) marks the Blacklist challenges
  ([Chapter 54](../C54-GameFlow-Blacklist/03-the-blacklist.md)) — the boss races.
- **Winning events** feeds bounty and milestones ([C54.4](../C54-GameFlow-Blacklist/04-bounty-milestones.md)) —
  progression.
- **The AI racers** ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md), `AIGoalRacer`) are your
  opponents.
- **The world** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) is the course, loaded per
  `GameFlowStates` ([C54.1](../C54-GameFlow-Blacklist/01-gameflow-states.md)).

So an event is a *node* where many systems converge — the career selects it, `GameFlowStates` loads it, `RaceFlow`
runs it, the sim ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) and AI
([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) drive it, and the result feeds progression
([C54.4](../C54-GameFlow-Blacklist/04-bounty-milestones.md)). Reading the events shows how the *structure*
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) and the *gameplay*
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)+) meet — an event is the unit where "the career"
becomes "a race you drive."

## RE implications

- **Anchor on** the `RaceFlow` states, the mode strings, the checkpoint system, and the `EVENT_*` types.
- **The RE workflow** — trace `RaceFlow` → identify the modes → map the checkpoints → follow the event types.
- **One flow, six modes** — the shared-framework pattern; each mode is *just its win condition*.
- **Events connect the career to the sim** — the node where structure meets gameplay.

---

### Key takeaways

- The events are anchored on the **`RaceFlow` states**, the **mode strings** (Circuit/Sprint/Drag/Speedtrap/
  Tollbooth/Knockout), the **checkpoint system**, and the **`EVENT_*` types**.
- The RE workflow: **trace `RaceFlow` → identify the modes → map the checkpoints → follow the event types**.
- The pattern is **one flow, six modes** — a shared `RaceFlow` machine with per-mode **win conditions** — the
  data-over-code/composition pattern; six event types for barely more than one.
- **`EVENT_RIVAL`** marks the **Blacklist challenges** ([Chapter 54](../C54-GameFlow-Blacklist/03-the-blacklist.md));
  winning events feeds **bounty and milestones**.
- An event is the **node where systems converge** — the career selects it, `GameFlowStates` loads it, `RaceFlow`
  runs it, the sim and AI drive it, the result feeds progression — where **structure meets gameplay**.

**Next:** [Chapter 56 — Customization: Performance & Visual](../C56-Customization/C56-Customization.md): the cars you
race with.

**Sources:** `speed.exe` (verified: `RaceCountdown`/`RaceActivity`/`RaceCompleted`/`RaceFinished`/`RaceAbandoned`;
mode strings `CircuitMenu`/`SprintMenu`/`DragMenu`/`DragTachometer`/`Dragster`/`speedtrap`/`speedtrapjump`/
`TollboothMarker`/`TollboothStat`/`KnockoutRacer`; `Checkpoint`/`CheckpointReached`/`checkpointrace`; `EVENT_RIVAL`/
`EVENT_ICON`/`EVENT_FENG`).
