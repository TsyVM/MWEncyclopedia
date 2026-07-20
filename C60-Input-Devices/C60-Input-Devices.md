# Chapter 60 — Input Devices & Control Mapping

> **Goal of this chapter:** decode how the player commands the car — the DirectInput devices (pad, wheel,
> keyboard), the abstract `GAME_ACTION_*` verbs (GAS, BRAKE, STEER, NOS, SHIFT…), and `InputPlayer` that maps
> device inputs to those verbs, feeding the car's INPUT mechanic.

The AI drives cars by planning ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)); the *player* drives
by *input* ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)). This chapter decodes the input path: the physical
devices (via DirectInput), the abstract game actions they map to, and the `InputPlayer` that bridges them to the
car. It's the player's half of the driver system — how a trigger-pull or a wheel-turn becomes throttle and steer
on the `AIVehicleHuman` ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)).

> **Verified against the executable.** The input system is named in `speed.exe`: **`InputPlayer`** and
> **`InputPlayerDrag`** (the player controllers); the **abstract `GAME_ACTION_*` verbs** (10) — `GAME_ACTION_GAS`,
> `GAME_ACTION_BRAKE`, `GAME_ACTION_HANDBRAKE`, `GAME_ACTION_STEERLEFT`/`STEERRIGHT`, `GAME_ACTION_NOS`,
> `GAME_ACTION_SHIFTUP`/`SHIFTDOWN`, `GAME_ACTION_RESET`, `GAME_ACTION_GAMEBREAKER`; the **devices** —
> `DirectInput`, `SteeringWheelDevice`, `Controller`/`ControllerUnplugged`; plus `Input`, `InputObject`,
> `InputNIS`.

---

## Deep-dive pages

- [C60.1 — The input model](01-input-model.md): device → `GAME_ACTION_*` → INPUT mechanic.
- [C60.2 — The game actions](02-game-actions.md): the 10 abstract command verbs.
- [C60.3 — Devices](03-devices.md): DirectInput, the wheel, pad, and keyboard.
- [C60.4 — InputPlayer](04-inputplayer.md): mapping devices to actions; the drag variant.
- [C60.5 — Reading input in RE](05-reading-input.md): navigating the input system.

---

## 60.1 The input model

The input path ([C60.1](01-input-model.md)) is a clean abstraction: **device → `GAME_ACTION_*` verb → INPUT
mechanic → controls**. A physical input (a trigger, a key) maps to an *abstract* game action
(`GAME_ACTION_GAS`), which the car's INPUT mechanic ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) reads as
throttle. The abstraction means the *same* game action (`GAME_ACTION_GAS`) can come from *any* device — a pad
trigger, a wheel pedal, a key — so the car doesn't care which device is used; it just reads the action.

## 60.2 The game actions

The **`GAME_ACTION_*` verbs** ([C60.2](02-game-actions.md)) are the abstract command vocabulary — 10 of them:
driving (`GAS`, `BRAKE`, `HANDBRAKE`, `STEERLEFT`/`STEERRIGHT`), performance (`NOS`, `SHIFTUP`/`SHIFTDOWN`), and
utility (`RESET` — respawn via `ResetCar` [C42.1](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md),
`GAMEBREAKER` — trigger a pursuit breaker [C49.5](../C49-Cops-Dispatch-Roadblocks/05-spikes-breakers.md)). These
are the *complete set* of things the player can command — the game's control surface, abstracted from any device.

## 60.3 Devices

The physical **devices** ([C60.3](03-devices.md)) are accessed via **`DirectInput`** — the DirectX input API of
the era. `SteeringWheelDevice` is the racing wheel (with pedals, and force feedback); a `Controller` is a gamepad;
the keyboard is another device. `ControllerUnplugged` handles a device being disconnected. Each device presents
its inputs (axes, buttons) which `InputPlayer` ([C60.4](04-inputplayer.md)) maps to the game actions
([C60.2](02-game-actions.md)) — so the wheel's pedal, the pad's trigger, and the key all map to `GAME_ACTION_GAS`.

## 60.4 InputPlayer

**`InputPlayer`** ([C60.4](04-inputplayer.md)) is the mapper — it reads the active device
([C60.3](03-devices.md)), applies the control mapping and response shaping (deadzones, sensitivity), and produces
the `GAME_ACTION_*` values ([C60.2](02-game-actions.md)) for the car's INPUT mechanic
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)). **`InputPlayerDrag`** is the variant for the drag minigame
([C55.3](../C55-Race-Events/03-drag.md)) — a different mapping (steering automated, focus on shift timing). So
`InputPlayer` is the player's counterpart to the AI planner ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)):
both produce the car's commands, one from a device, one from a plan.

---

### Key takeaways

- The input path is **device → `GAME_ACTION_*` verb → INPUT mechanic → controls** — an abstraction so the car reads
  *actions*, not devices.
- The **10 `GAME_ACTION_*` verbs** are the abstract command vocabulary — driving (GAS/BRAKE/HANDBRAKE/STEER),
  performance (NOS/SHIFT), utility (RESET/GAMEBREAKER).
- **Devices** (via `DirectInput`) — `SteeringWheelDevice` (wheel), `Controller` (pad), keyboard — each maps its
  inputs to the shared game actions.
- **`InputPlayer`** maps the active device to game actions (with deadzones/sensitivity); **`InputPlayerDrag`** is
  the drag-mode variant.
- `InputPlayer` is the **player's counterpart to the AI planner** — both produce the car's commands, one from a
  device, one from a plan ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)).

**Next:** [Chapter 61 — Traffic & Ambient World Life](../C61-Traffic-Ambient/C61-Traffic-Ambient.md): the cars that
fill the city.
