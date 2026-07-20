# C55.2 — Circuit & Sprint

> **The one-sentence version:** the two core road races are Circuit (multi-lap around a closed loop) and Sprint (a
> one-way dash from start to finish) — both wheel-to-wheel against AI racers through checkpoints, differing only in
> course shape (repeated lap vs. point-to-point).

[← C55.1 — The RaceFlow machine](01-race-flow.md) · [Chapter 55 hub](C55-Race-Events.md) ·
[Next: C55.3 — Drag →](03-drag.md)

---

## The two road races

The heart of Most Wanted's racing is the two **road-race** modes — head-to-head races against AI opponents through
the city streets:

- **Circuit** (`CircuitMenu`) — a **multi-lap** race around a **closed loop**. You do several laps of the same
  circuit; first across the line after the final lap wins.
- **Sprint** (`SprintMenu`) — a **one-way** race from a start point to a finish point across the city; first to the
  finish wins. No laps — a single point-to-point dash.

Both are **wheel-to-wheel** races against AI racers ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md),
the `AIGoalRacer` brain, [C46.4](../C46-AI-Goals-Actions/04-override-goals.md)) through the checkpoint course
([C55.1](01-race-flow.md)). They're the modes where *racing skill* — line, overtaking, defending — matters most,
and they make up the bulk of the career's events ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).

> ✅ *Verified:* `CircuitMenu`, `SprintMenu`, and the checkpoint system (`Checkpoint`/`CheckpointReached`,
> `checkpointrace`) are present in `speed.exe`. Both run the `RaceFlow` machine ([C55.1](01-race-flow.md)) against
> AI racers ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)).

## Circuit: the lap race

**Circuit** is the classic *lap race*:

- **A closed loop** — the course returns to its start, so you drive it multiple times (typically 2–4 laps).
- **Lap counting** — each time you complete the loop (pass the start/finish checkpoint,
  [C55.1](01-race-flow.md)), a lap is counted; the race ends after the set number.
- **Repetition rewards learning** — because you repeat the loop, you *learn* it — the braking points, the racing
  line, the shortcuts — and get faster each lap. Circuit rewards *mastery* of a course.

So Circuit is about *consistency and learning* — driving the same loop well, repeatedly, and using your growing
familiarity to build a lead. The multi-lap structure also creates *comeback* opportunities (a mistake isn't
fatal — there's another lap) and *tactical* racing (managing a lead or hunting a rival over several laps). It's the
most "traditional racing" of MW's modes.

## Sprint: the point-to-point dash

**Sprint** is the *one-shot* race:

- **Start to finish** — a single route across the city, from A to B, no repetition.
- **One chance** — there are no laps, so *every moment counts*; a single mistake can lose the race with no lap to
  recover.
- **Route knowledge over learning** — you can't learn the course by repetition (you drive it once), so *knowing the
  city* ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) and reading the road ahead matter more.

So Sprint is about *nerve and precision* — one flawless run from start to finish, with no second chances. It's more
*tense* than Circuit (no recovery) and rewards *aggression* (you must commit) and *city knowledge* (you can't
pre-learn the specific route by lapping). The point-to-point format also lets Sprints traverse *dramatic* routes
across the whole map ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — a Sprint can be a long,
scenic dash the way a Circuit (a fixed loop) can't.

> 🟡 *Reasoned:* the characterisations of Circuit (learning/consistency) and Sprint (nerve/route-knowledge) follow
> from their lap vs. point-to-point structure; the exact lap counts and routes are per-event data. The modes and the
> checkpoint system are verified.

## Same flow, different shape

The elegance is that Circuit and Sprint are the *same race* ([C55.1](01-race-flow.md)) — same countdown, same
wheel-to-wheel activity, same finish — differing only in **course shape**:

- **Circuit** = a checkpoint loop, traversed N times (lap counting).
- **Sprint** = a checkpoint line, traversed once (finish at the end).

Both are just *a sequence of checkpoints* ([C55.1](01-race-flow.md)) — Circuit's sequence loops back and repeats;
Sprint's runs straight to a finish. So the *only* real difference is whether the checkpoint course is a loop or a
line, and whether laps are counted. This is why they share almost all their code — they're one race mode
parameterised by course topology. The `RaceFlow` machine ([C55.1](01-race-flow.md)) runs both; the course data
(loop vs. line) makes them Circuit or Sprint. It's the data-over-code pattern
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) applied to race modes.

## RE implications

- **Circuit** (multi-lap loop) and **Sprint** (point-to-point) are the two core road races vs. AI racers.
- **Circuit** rewards *learning/consistency* (repeated laps); **Sprint** rewards *nerve/route-knowledge* (one shot).
- **Same `RaceFlow`, different course shape** — a checkpoint loop (Circuit) vs. a checkpoint line (Sprint).
- **Data-over-code** — the course topology makes a race Circuit or Sprint; the flow is shared.

---

### Key takeaways

- **Circuit** (`CircuitMenu`, multi-lap closed loop) and **Sprint** (`SprintMenu`, one-way point-to-point) are the
  two core **road races** — wheel-to-wheel against AI racers through checkpoints.
- **Circuit** rewards **learning and consistency** — you lap the same loop, get faster, and race tactically over
  several laps (mistakes recoverable).
- **Sprint** rewards **nerve and city knowledge** — one flawless run start-to-finish, no laps to recover, across
  dramatic routes.
- They're the **same race** differing only in **course shape** — a checkpoint *loop* (Circuit, laps counted) vs. a
  checkpoint *line* (Sprint) — the data-over-code pattern applied to modes.
- Together they're the **bulk of the career's racing** — where racing skill (line, overtaking) matters most.

**Continue:** [C55.3 — Drag](03-drag.md) · [Chapter 55 hub](C55-Race-Events.md)
