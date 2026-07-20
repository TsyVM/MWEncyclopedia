# C60.2 — The Game Actions

> **The one-sentence version:** the 10 `GAME_ACTION_*` verbs are the player's complete abstract command vocabulary
> — driving (GAS, BRAKE, HANDBRAKE, STEERLEFT/RIGHT), performance (NOS, SHIFTUP/DOWN), and utility (RESET,
> GAMEBREAKER).

[← C60.1 — The input model](01-input-model.md) · [Chapter 60 hub](C60-Input-Devices.md) ·
[Next: C60.3 — Devices →](03-devices.md)

---

## The command vocabulary

The `GAME_ACTION_*` verbs are the *complete set* of things the player can command — the game's control surface,
abstracted from any device ([C60.1](01-input-model.md)). The 10 verified verbs group into three:

**Driving (5):**

| Action | Command |
|---|---|
| `GAME_ACTION_GAS` | throttle ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `GAME_ACTION_BRAKE` | brake |
| `GAME_ACTION_HANDBRAKE` | handbrake (initiate a drift/slide, [C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) |
| `GAME_ACTION_STEERLEFT` / `GAME_ACTION_STEERRIGHT` | steering |

**Performance (3):**

| Action | Command |
|---|---|
| `GAME_ACTION_NOS` | fire nitrous ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `GAME_ACTION_SHIFTUP` / `GAME_ACTION_SHIFTDOWN` | manual gear change ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |

**Utility (2):**

| Action | Command |
|---|---|
| `GAME_ACTION_RESET` | respawn the car (`ResetCar`, [C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)) |
| `GAME_ACTION_GAMEBREAKER` | trigger a pursuit breaker ([C49.5](../C49-Cops-Dispatch-Roadblocks/05-spikes-breakers.md)) |

So the whole game — driving, boosting, shifting, resetting, breaking pursuits — is these 10 actions. Nothing the
player does isn't one of them.

> ✅ *Verified:* the 10 `GAME_ACTION_*` verbs — `GAS`, `BRAKE`, `HANDBRAKE`, `STEERLEFT`, `STEERRIGHT`, `NOS`,
> `SHIFTUP`, `SHIFTDOWN`, `RESET`, `GAMEBREAKER` — are present in `speed.exe`.

## Analog vs. digital actions

The actions divide by *type* — some are **analog** (continuous), some **digital** (on/off):

- **Analog** — `GAS`, `BRAKE`, `STEERLEFT`/`STEERRIGHT` are *continuous* (0..1) — a trigger half-pulled is half
  throttle, a wheel turned partway is partial steer. These need the precision of an analog device
  ([C60.3](03-devices.md)) or are approximated on digital ones (a key is full-on).
- **Digital** — `NOS`, `SHIFTUP`/`SHIFTDOWN`, `RESET`, `GAMEBREAKER`, `HANDBRAKE` are *events/toggles* — you fire
  NOS or you don't; you shift up once per press. A button suffices.

So the action set spans continuous control (the driving) and discrete events (the performance/utility). This
matters for devices ([C60.3](03-devices.md)): a wheel's analog pedals and axis give precise `GAS`/`STEER`, while a
keyboard's keys give binary approximations — which is why a wheel or analog pad *drives better* (finer analog
control) than a keyboard, even though both map to the same actions. The action *type* (analog/digital) determines
how well each device can express it.

> 🟡 *Reasoned:* the analog/digital classification of the actions follows from their meaning (throttle/steer are
> continuous, NOS/shift are events), consistent with the verified action set and the device types
> ([C60.3](03-devices.md)); the exact per-action value ranges are per-config. The action names are verified.

## GAMEBREAKER and RESET: the utility verbs

Two actions are worth noting as *not* pure driving — they command *game systems*:

- **`GAME_ACTION_GAMEBREAKER`** triggers a **pursuit breaker** ([C49.5](../C49-Cops-Dispatch-Roadblocks/05-spikes-breakers.md))
  — the droppable environment that disables cops. That this is a *first-class game action* (not just driving into
  the object) confirms breakers are a *deliberate player tool* ([C49.5](../C49-Cops-Dispatch-Roadblocks/05-spikes-breakers.md))
  — you *activate* them, they're part of your control surface. The "gamebreaker" naming is evocative: it *breaks*
  the pursuit's momentum.
- **`GAME_ACTION_RESET`** respawns the car via `ResetCar` ([C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md))
  — recovering from a flip or a dead-end. A utility for keeping play flowing.

So the action set includes not just *driving* the car but *using the game's tools* (breakers) and *recovering*
(reset). This reflects that Most Wanted is more than a driving sim — it's a pursuit game where triggering breakers
and resetting are core interactions, elevated to first-class actions alongside the throttle. The 10 actions are the
*complete* player interface: drive, boost, shift, reset, break.

## Why a fixed action set

Defining a *fixed, small* set of abstract actions ([above](#the-command-vocabulary)) is clean design:

- **A complete, closed interface.** 10 actions cover everything; the game logic
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md),
  [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) reads only these, so the input
  surface is bounded and known.
- **The remapping target.** The actions are what the player *rebinds to* ([C60.1](01-input-model.md)) — a fixed
  set of meanings that any device input can be assigned to.
- **The AI equivalent.** The AI produces the *same* commands ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md))
  — the goal/action system ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) outputs throttle/steer
  equivalent to `GAS`/`STEER`, so the car's driving model reads one command set from either source.

So the `GAME_ACTION_*` set is the *contract* between the player (or AI) and the car — a fixed, complete vocabulary
of commands. Everything the player can do is one of these 10 verbs, mapped from a device
([C60.4](04-inputplayer.md)); everything the car responds to is one of these 10, read by its INPUT mechanic
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)). The action set is the still point at the centre of the input
system.

## RE implications

- **10 `GAME_ACTION_*` verbs** are the complete player command set — driving (5), performance (3), utility (2).
- **Analog vs. digital** — `GAS`/`BRAKE`/`STEER` are continuous; `NOS`/`SHIFT`/`RESET`/`GAMEBREAKER` are events —
  determining how well each device expresses them.
- **`GAMEBREAKER`/`RESET`** are first-class game-system verbs — breakers are a deliberate player tool.
- **A fixed action set** is the contract between player/AI and the car — a complete, closed command vocabulary.

---

### Key takeaways

- The **10 `GAME_ACTION_*` verbs** are the player's complete command vocabulary — **driving** (GAS, BRAKE,
  HANDBRAKE, STEERLEFT/RIGHT), **performance** (NOS, SHIFTUP/DOWN), **utility** (RESET, GAMEBREAKER).
- Actions split **analog** (GAS/BRAKE/STEER — continuous) vs. **digital** (NOS/SHIFT/RESET/GAMEBREAKER — events) —
  why a wheel/analog pad drives more precisely than a keyboard.
- **`GAME_ACTION_GAMEBREAKER`** and **`GAME_ACTION_RESET`** are first-class **game-system** verbs — pursuit
  breakers are a deliberate player tool; reset keeps play flowing.
- The action set is **fixed and complete** — the contract between player/AI and the car; the AI produces the same
  commands ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)).
- Everything the player does is **one of these 10 verbs** — the still point at the centre of the input system.

**Continue:** [C60.3 — Devices](03-devices.md) · [Chapter 60 hub](C60-Input-Devices.md)
