# C47.2 — The Player Is an AI

> **The one-sentence version:** the player's car is an `AIVehicleHuman` (vtable `0x0089293C`, 133 methods) — it
> runs the *same* driver-brain machinery as every AI car, but takes its commands from `InputPlayer` instead of a
> goal/action planner, which is why the registry has no separate "player vehicle" class.

[← C47.1 — The AIVehicle hierarchy](01-aivehicle-hierarchy.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md) ·
[Next: C47.3 — The managers →](03-managers.md)

---

## The player car is an AIVehicle

The single most illuminating fact about the AI architecture: **the player's car is an `AIVehicle`** — specifically
`AIVehicleHuman` (verified vtable `0x0089293C`, 133 methods, 2028 B). The player's car is not a special case
outside the AI system; it's a *member* of the driver-brain family ([C47.1](01-aivehicle-hierarchy.md)), sharing the
base `AIVehicle`'s machinery:

- **Same brain wiring.** `AIVehicleHuman` runs the shared `AIVehicle` machinery
  ([C47.1](01-aivehicle-hierarchy.md)) — traction assists, reset handling, the bridge from abstract inputs to
  physics ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
- **Different command source.** Where an AI brain takes commands from a goal/action planner
  ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), `AIVehicleHuman` takes them from **`InputPlayer`**
  ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) — the player's controller.
- **No separate player class.** Because the player's car is an `AIVehicleHuman`, there is *no distinct "player
  vehicle" class* in the registry. The player is a *kind of AI vehicle*.

> ✅ *Verified:* `AIVehicleHuman` is a real vtable at `0x0089293C` with 133 methods (2028 B), a member of the
> `AIVehicle*` family sharing the base machinery ([C47.1](01-aivehicle-hierarchy.md)). Hash `0x73A9594F`.

## The unification, at the class level

This is the **class-level counterpart** to the input/AI mechanic swap ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)):

- **At the mechanic level** ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)): a car has an INPUT mechanic
  (player) *or* an AI mechanic (planner) — the driver is swappable.
- **At the class level** (this page): the player's car is `AIVehicleHuman` and a cop is `AIVehicleCopCar`, both
  `AIVehicle` subclasses — the driver *brain* is the same base, specialised.

Both express one deep design decision: **the player and the AI are the same kind of driver.** Everything on the
road — the player, cops, racers, traffic — is an `AIVehicle` running the same fundamental machinery, differing only
in where its commands come from (player input vs. a planner) and which slices of behaviour it overrides. This
unification is why the game world is coherent: the player's car obeys the same physics and the same driver-brain
plumbing as every other car, so the player and the AI meet on equal terms.

## Why the player has 133 methods but 2028 bytes

`AIVehicleHuman` has *few* overrides (133 methods, the fewest of the family) but *large* state (2028 B) — the
second-largest. This apparent paradox is explained by "**a lot of state, mostly driven from outside**":

- **Few overrides.** The player brain doesn't need to *plan* — no goal/action selection
  ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), because the player decides. So it overrides
  fewer of the base's 351 methods ([C47.1](01-aivehicle-hierarchy.md)) — mostly the command-source and assist parts.
- **Large state.** But it still carries the full driver state — perception, physics bridge, assists, reset
  context, telemetry — because it's still a complete driver brain, just commanded externally.

So the player car is "a lot of machinery, lightly overridden": it keeps almost all of the base `AIVehicle`'s
apparatus (hence the size) but replaces little of its behaviour (hence the low method count), because the *decision*
part is outsourced to the human. This is the signature of a class that's *configured* rather than *specialised* —
the player brain is the base brain with the planner replaced by `InputPlayer`.

## The brain's state

The `AIVehicle` base state (shared by all brains, including the player's) has verified named offsets — a window
into what a driver brain tracks:

| Offset | Field | Meaning |
|---|---|---|
| `+0x38` | `mDriveSpeed` | the brain's current target drive speed |
| `+0x84` | `mTopSpeed` | the car's top speed |
| `+0x6C4` | `mAccelerationTableData[10]` | a 10-entry acceleration table |
| `+0x6EC` | `mAccelerationMaxSpeed` | the acceleration model's max speed |

These are the *driving model* the brain uses to pace the car — a target speed (`mDriveSpeed`), the car's limits
(`mTopSpeed`), and an acceleration curve (`mAccelerationTableData[10]`). Every driver brain, AI or human, tracks
these; for an AI they're set by the action ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), for the
player they follow the input. The 1852+ bytes of an `AIVehicle` are full of such fields — perception, targets,
timers, route context — the working memory of a driver.

> ✅ *Verified:* the `AIVehicle` base offsets `+0x38 mDriveSpeed`, `+0x84 mTopSpeed`, `+0x6C4
> mAccelerationTableData[10]`, `+0x6EC mAccelerationMaxSpeed` are recovered from the disassembly — the brain's
> driving-model state.

## RE implications

- **The player's car is `AIVehicleHuman`** — the same `AIVehicle` machinery, commanded by `InputPlayer` not a
  planner; no separate player class.
- **The class-level unification** — player and AI are the same kind of driver
  ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) — the world is coherent because all cars share the brain.
- **Few overrides, large state** — the player brain is *configured* (planner → input), not specialised; it keeps
  the machinery, replaces the decision.
- **The brain's state** (`mDriveSpeed`, `mTopSpeed`, `mAccelerationTableData`) is the driving model every brain
  tracks.

---

### Key takeaways

- **The player's car is an `AIVehicleHuman`** (verified vtable `0x0089293C`, 133 methods) — the same driver-brain
  machinery as every AI car, commanded by **`InputPlayer`** instead of a planner.
- There is **no separate "player vehicle" class** — the player is a *kind of AI vehicle*, the class-level
  unification of player and AI.
- The paradox of **few overrides (133) but large state (2028 B)** reflects "a lot of machinery, lightly
  overridden" — the decision is outsourced to the human, the apparatus is kept.
- The `AIVehicle` base tracks a **driving model** — verified offsets `+0x38 mDriveSpeed`, `+0x84 mTopSpeed`,
  `+0x6C4 mAccelerationTableData[10]`, `+0x6EC mAccelerationMaxSpeed`.
- This unification makes the world **coherent** — player and AI meet on equal terms, sharing the brain and the
  physics.

**Continue:** [C47.3 — The managers](03-managers.md) · [Chapter 47 hub](C47-AI-Driver-Vehicle.md)
