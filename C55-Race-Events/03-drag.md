# C55.3 — Drag

> **The one-sentence version:** Drag is a straight-line race where the skill is shifting, not steering — the
> `DragTachometer` shows the perfect-shift window, steering is automated (stay in lane), and `EngineDragster` is
> the specialised drivetrain built for the launch and the redline.

[← C55.2 — Circuit & Sprint](02-circuit-sprint.md) · [Chapter 55 hub](C55-Race-Events.md) ·
[Next: C55.4 — Speedtrap, Tollbooth & Knockout →](04-speed-modes.md)

---

## A different game

Drag (`DragMenu`) is the odd one out — it's not really a *driving* race, it's a *shifting* minigame. In a drag
race:

- **The course is a straight line** — a wide, straight stretch; no corners to negotiate.
- **Steering is automated** — you don't steer to stay on the road; the car holds its lane
  ([reasoned](#re-implications)). You *do* change lanes (to dodge traffic) but not steer through turns.
- **The skill is shifting** — launching perfectly and hitting the perfect-shift window at each gear change
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) is what wins.
- **Speeds are extreme** — top gears at full tune, weaving through traffic at terrifying speed.

So Drag distills driving down to *the drivetrain* — the launch, the shifts, the redline
([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) — plus split-second traffic dodging. It's a
test of *timing and nerve* rather than *racing line*, a completely different skill from Circuit/Sprint
([C55.2](02-circuit-sprint.md)).

> ✅ *Verified:* `DragMenu`, `DragTachometer`, `Drag`, and `Dragster`/`EngineDragster`
> ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) are present in `speed.exe` — the drag mode,
> its tachometer UI, and its specialised drivetrain.

## The DragTachometer

The signature UI of Drag is the **`DragTachometer`** (verified) — a prominent rev meter that's the *interface* of
the shifting game:

- **The rev needle** — shows engine RPM ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md))
  climbing toward the redline.
- **The perfect-shift window** — a marked zone near the redline where shifting gives a boost (a perfect shift
  keeps you in the power band, [C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)); shift too early
  and you lose momentum, too late and you hit the limiter.
- **The launch** — off the line, a perfect launch (clutch/rev, [C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md))
  sets your start.

So the `DragTachometer` is *the game* in Drag — you watch it, not the road, timing each shift to the perfect
window. This is why Drag has a *dedicated* tachometer UI ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)):
the shifting is the core mechanic, so it gets a prominent, precise readout. The perfect-shift window
([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) is what turns raw acceleration into a *skill*
— anyone can hold the throttle, but hitting every perfect shift is the mark of a good drag racer.

## EngineDragster: the specialised drivetrain

Drag uses a *specialised* drivetrain — **`EngineDragster`** ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md),
132 methods, the largest engine class) — because the drag context needs mechanics the road races don't:

- **The launch** — the clutch-dump / rev-limiter launch off the line, which road races don't emphasise.
- **The perfect-shift window** — the boost mechanic ([above](#the-dragtachometer)) tied to precise shift timing.
- **The redline management** — extreme-RPM behaviour at the top of each gear.

That `EngineDragster` is the *most-methoded* engine class ([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md))
reflects this specialisation — the launch and perfect-shift mechanics add code the standard `EngineRacer`
([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) doesn't need. So Drag isn't just a mode with
different *rules* — it swaps in a different *drivetrain* tuned for the drag game. This is the mechanic-swapping model
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) at the mode level: a drag car uses `EngineDragster` where a
circuit car uses `EngineRacer`, the same chassis with a mode-appropriate engine mechanic.

> 🟡 *Reasoned:* the automated-steering and lane-dodging characterisation of Drag is the mode's documented design;
> the exact steering-assist implementation is deeper RE. The mode, the `DragTachometer`, and `EngineDragster` (132
> methods) are verified.

## Why a shifting minigame

Including a pure *shifting* mode (rather than only steering races) diversifies the skill set and showcases the
drivetrain:

- **A different skill.** Drag tests *timing* (launch, shifts) where Circuit/Sprint test *steering* (line,
  overtaking) — variety in what the player masters.
- **Showcases the powertrain.** Drag foregrounds the engine/drivetrain ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md))
  — the launch, the shifts, the power curve — the systems that are *background* in a steering race become the *whole
  game*.
- **High-tension moments.** Extreme speed + traffic-dodging + perfect-shift timing = intense, short bursts — a
  different rhythm from the sustained racing of Circuit.

So Drag is the *drivetrain minigame* — a mode built to spotlight the shifting and launch mechanics
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) that the other modes use but
don't foreground. It's a nice example of the engine's flexibility: the same car, in Drag, becomes a shifting game by
swapping the drivetrain (`EngineDragster`) and the UI (`DragTachometer`) — a distinct experience built from
mode-appropriate components on the shared `RaceFlow` ([C55.1](01-race-flow.md)).

## RE implications

- **Drag** is a **straight-line shifting** minigame — automated steering, the skill is the launch and perfect
  shifts.
- **`DragTachometer`** is the core UI — the rev meter with the perfect-shift window
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)).
- **`EngineDragster`** (132 methods, largest engine) is the specialised drivetrain — launch, perfect-shift, redline.
- **A drivetrain minigame** — swaps the engine mechanic and UI to spotlight shifting, on the shared `RaceFlow`.

---

### Key takeaways

- **Drag** (`DragMenu`) is a **straight-line shifting minigame** — steering is automated (stay in lane, dodge
  traffic); the skill is the **launch and perfect shifts**.
- The **`DragTachometer`** is the core UI — the rev meter with the **perfect-shift window** near the redline
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)); you watch it, not the road.
- Drag uses the specialised **`EngineDragster`** drivetrain (**132 methods**, the largest engine class) — built for
  the launch, perfect-shift boost, and redline the road races don't emphasise.
- It's the **mechanic-swap model at the mode level** — a drag car swaps in `EngineDragster` + `DragTachometer`,
  same chassis, different game.
- Drag **spotlights the drivetrain** ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md))
  — the shifting/launch mechanics that are background in steering races become the whole game.

**Continue:** [C55.4 — Speedtrap, Tollbooth & Knockout](04-speed-modes.md) · [Chapter 55 hub](C55-Race-Events.md)
