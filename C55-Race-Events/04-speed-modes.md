# C55.4 — Speedtrap, Tollbooth & Knockout

> **The one-sentence version:** three alternative-scoring modes reward different skills — Speedtrap scores your
> cumulative speed through cameras, Tollbooth is a time trial against the clock to each marker, and Lap Knockout
> eliminates the last-place racer each lap — each reusing the `RaceFlow` machine with a different win condition.

[← C55.3 — Drag](03-drag.md) · [Chapter 55 hub](C55-Race-Events.md) ·
[Next: C55.5 — Reading events in RE →](05-reading-events.md)

---

## Different win conditions

Beyond the head-to-head road races ([C55.2](02-circuit-sprint.md)) and the drag minigame
([C55.3](03-drag.md)), three modes change the **win condition** — the same `RaceFlow`
([C55.1](01-race-flow.md)) with a different definition of winning:

- **Speedtrap** (`speedtrap`) — win by *speed*, not position.
- **Tollbooth** (`TollboothMarker`) — win by *beating the clock*, not rivals.
- **Lap Knockout** (`KnockoutRacer`) — win by *not being last*, each lap.

These reward *different skills* than pure racing — top speed, time management, consistency — so the career
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) tests a *variety* of abilities, not just
wheel-to-wheel racecraft. Each reuses the shared framework ([C55.1](01-race-flow.md)) with its own scoring.

> ✅ *Verified:* `speedtrap`/`speedtrapjump`, `TollboothMarker`/`TollboothStat`, and `KnockoutRacer`/`knockout` are
> present in `speed.exe` — the three alternative-scoring modes; all run the `RaceFlow` machine
> ([C55.1](01-race-flow.md)).

## Speedtrap: score by speed

**Speedtrap** scores your **speed through cameras** along the route:

- **Speed cameras** are placed along the course (the "traps"); each records your speed as you pass.
- **Cumulative speed wins** — your trap speeds are summed (or averaged), and the highest total wins — *not* first
  place.
- **The tension** — you must be *fastest at the traps*, so you push for top speed *through* each camera, which may
  mean different driving than racing for position (flat-out through the trap, not braking for the racing line).

So Speedtrap rewards *raw speed* over *positioning* — a car (and driver) tuned for top speed
([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) excels, and the skill is carrying maximum
speed through each trap. It's a different optimisation from a race: you're not fighting rivals for position, you're
fighting the *speedometer* at each camera. `speedtrapjump` suggests trap-related events/jumps
([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)). This mode showcases *top-end performance* — the reason to build
a fast car ([Chapter 56](../C56-Customization/C56-Customization.md)).

## Tollbooth: race the clock

**Tollbooth** is a **time trial** — you race the *clock*, not other cars:

- **A chain of tollbooths** (`TollboothMarker`) along a route, each with a time limit.
- **Reach each in time** — passing a tollbooth *adds time*; the goal is to reach each before the clock hits zero,
  chaining through all of them.
- **No rivals** — it's just you and the timer; a mistake (a crash, a wrong turn) costs time you may not recover.

So Tollbooth is about *sustained pace and precision* — a flawless, fast run through a sequence of checkpoints
([C55.1](01-race-flow.md)) against a ticking clock. It's tense in a different way from a race: there's no rival to
beat, just the *unforgiving clock*. Every second matters, so it rewards *clean, committed* driving — no mistakes,
maximum pace. `TollboothStat` tracks your performance. This mode tests *consistency under time pressure* — can you
string together a perfect run?

## Lap Knockout: don't be last

**Lap Knockout** (`KnockoutRacer`) is an *elimination* race:

- **Multiple laps** (like Circuit, [C55.2](02-circuit-sprint.md)) with several racers.
- **Last place eliminated each lap** — whoever is last when the lead car completes a lap is *knocked out*.
- **Survive to win** — racers are eliminated lap by lap until one remains — the winner.

So Knockout rewards *not making mistakes* and *avoiding last place* — you don't have to *win* each lap, just not be
*last*. This creates a distinct tension: the fight is at the *back* (avoiding elimination), not just the front. A
consistent racer who never drops to last outlasts a fast-but-erratic one who occasionally falls behind. `KnockoutRacer`
is the AI racer ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) in this mode. It tests
*consistency and race-craft under elimination pressure* — a survival race.

> 🟡 *Reasoned:* the win-condition characterisations (Speedtrap cumulative speed, Tollbooth time-chain, Knockout
> last-place elimination) are Most Wanted's documented mode designs, consistent with the verified mode strings and
> the shared `RaceFlow`; the exact scoring formulas are per-mode data. The modes are verified present.

## Why alternative modes

Adding alternative-scoring modes (rather than only head-to-head races) enriches the career
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)):

- **They test varied skills.** Top speed (Speedtrap), time management (Tollbooth), consistency (Knockout) — so the
  career rewards a *well-rounded* driver, not just a racer.
- **They reuse the framework.** Each is `RaceFlow` ([C55.1](01-race-flow.md)) + checkpoints
  ([C55.1](01-race-flow.md)) + a different win condition — cheap to build, since the flow is shared.
- **They vary the rhythm.** A career of only wheel-to-wheel races would be monotonous; interleaving speed, time, and
  elimination modes keeps it fresh.

So the alternative modes are the career's *variety* — different games on the same chassis
([C55.1](01-race-flow.md)), each a new way to test the player. Together with Circuit/Sprint
([C55.2](02-circuit-sprint.md)) and Drag ([C55.3](03-drag.md)), they give Most Wanted a *diverse* event roster —
the range of challenges that fill the climb up the Blacklist
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)), all built economically on the shared `RaceFlow`
machine.

## RE implications

- **Three alternative-scoring modes** — Speedtrap (cumulative speed), Tollbooth (beat the clock), Lap Knockout
  (last-place elimination).
- **Each is `RaceFlow` + a different win condition** — reusing the shared framework
  ([C55.1](01-race-flow.md)).
- **They test varied skills** — top speed, time management, consistency — for a well-rounded career.
- **Economical variety** — different games on the shared chassis, keeping the career fresh.

---

### Key takeaways

- Three **alternative-scoring** modes change the win condition on the shared `RaceFlow`: **Speedtrap** (cumulative
  **speed** through cameras), **Tollbooth** (beat the **clock** to each marker), **Lap Knockout** (**last-place
  eliminated** each lap).
- **Speedtrap** rewards **raw top speed** (fight the speedometer, not rivals); **Tollbooth** rewards **clean pace
  under a clock** (no rivals, just the timer); **Knockout** rewards **consistency** (don't be last).
- Each **reuses `RaceFlow` + checkpoints** with a different scoring — cheap to build on the shared framework
  ([C55.1](01-race-flow.md)).
- They **test varied skills** (speed, timing, consistency) — so the career rewards a **well-rounded** driver, not
  just a wheel-to-wheel racer.
- With Circuit/Sprint and Drag, they give MW a **diverse event roster** — economical variety filling the Blacklist
  climb.

**Continue:** [C55.5 — Reading events in RE](05-reading-events.md) · [Chapter 55 hub](C55-Race-Events.md)
