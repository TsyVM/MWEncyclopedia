# C60.5 — Reading Input in RE

> **The one-sentence version:** navigate the input system by the `GAME_ACTION_*` verbs, `InputPlayer`/
> `InputPlayerDrag`, and the `DirectInput` device classes — reading input as device → mapper → abstract actions →
> the car.

[← C60.4 — InputPlayer](04-inputplayer.md) · [Chapter 60 hub](C60-Input-Devices.md) ·
[Next: Chapter 61 — Traffic & Ambient World Life →](../C61-Traffic-Ambient/C61-Traffic-Ambient.md)

---

## Anchors for input RE

The input system is anchored on verified strings:

- **The game actions** — the 10 `GAME_ACTION_*` verbs ([C60.2](02-game-actions.md)).
- **The mapper** — `InputPlayer`, `InputPlayerDrag`, `InputObject` ([C60.4](04-inputplayer.md)).
- **The devices** — `DirectInput`, `SteeringWheelDevice`, `Controller`, `ControllerUnplugged`
  ([C60.3](03-devices.md)).

From these, the input system is navigable: the actions, the mapper, and the devices.

## The RE workflow

Reading input:

1. **Enumerate the actions** — grep `GAME_ACTION_*` ([C60.2](02-game-actions.md)); the complete command set.
2. **Trace the mapper** — `InputPlayer` reading devices and producing actions ([C60.4](04-inputplayer.md)).
3. **Map the devices** — the `DirectInput` device classes ([C60.3](03-devices.md)).
4. **Follow to the car** — the actions feeding the INPUT mechanic
   ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md), [C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)).

The output is the full input picture: actions, mapper, devices, and the path to the car.

## The GAME_ACTION_ set is the control spec

The single most useful RE artifact for input is the **`GAME_ACTION_*` set** ([C60.2](02-game-actions.md)) — grep it
and you have the game's *entire control specification* in one list:

```
GAME_ACTION_GAS / BRAKE / HANDBRAKE / STEERLEFT / STEERRIGHT   ← driving
GAME_ACTION_NOS / SHIFTUP / SHIFTDOWN                          ← performance
GAME_ACTION_RESET / GAMEBREAKER                                ← utility
```

These 10 verbs are *everything* the player can do — a complete, closed interface ([C60.2](02-game-actions.md)). So
reverse-engineering "what can the player control?" is a single grep. And because the actions are the *stable
meaning* ([C60.1](01-input-model.md)), tracing where each is *read* (in the car's INPUT mechanic,
[C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) tells you *what each does*. The action set is thus the input
system's *table of contents* — the self-documenting spec ([C50.2](../C50-Verification-Methodology/02-byte-verification.md))
of the game's controls, recoverable in one command.

## Input completes the player side

With input decoded, the *player's* side of the game is complete — mirroring the AI's:

| | Player | AI |
|---|---|---|
| **Command source** | device (`InputPlayer`, this chapter) | plan (goals/actions, [Ch 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) |
| **Command form** | `GAME_ACTION_*` | throttle/steer ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) |
| **Consumer** | INPUT mechanic ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) | AI mechanic ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) |
| **Vehicle** | `AIVehicleHuman` ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)) | `AIVehicleCopCar`/`Racecar` ([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md)) |

So the player and AI are *perfectly symmetric* — two command sources, one command interface, one driving model.
This symmetry ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)) is the elegant core of the driver system:
build one car (`AIVehicle`, [C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md)) that takes commands, and
supply the commands from either a human (this chapter) or a machine
([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)). Reading input closes this loop — the *human* half
of the command system, symmetric to the *AI* half, both feeding the shared car. The driver system
([Chapters 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)–47, 60) is now complete: brains, planners, and the
player's input, all producing commands for one car.

## RE implications

- **Anchor on** the `GAME_ACTION_*` verbs, `InputPlayer`/`InputPlayerDrag`, and the `DirectInput` device classes.
- **The RE workflow** — enumerate actions → trace the mapper → map the devices → follow to the car.
- **The `GAME_ACTION_*` set is the control spec** — grep it for the game's complete control surface.
- **Input completes the player side** — symmetric to the AI; both feed the shared car.

---

### Key takeaways

- The input system is anchored on the **`GAME_ACTION_*` verbs**, **`InputPlayer`**/`InputPlayerDrag`, and the
  **`DirectInput` device classes**.
- The RE workflow: **enumerate the actions → trace the mapper → map the devices → follow to the car**.
- The **`GAME_ACTION_*` set is the control spec** — a single grep gives the game's *complete, closed* control
  surface (10 verbs); tracing where each is read gives what it does.
- **Input completes the player side** — perfectly **symmetric to the AI**: two command sources (device vs. plan),
  one command interface (`GAME_ACTION_*`/throttle-steer), one driving model.
- The driver system (Chapters 46–47, 60) is now complete — **brains, planners, and player input**, all producing
  commands for the shared car ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)).

**Next:** [Chapter 61 — Traffic & Ambient World Life](../C61-Traffic-Ambient/C61-Traffic-Ambient.md): the cars that
fill the city.

**Sources:** `speed.exe` (verified: `GAME_ACTION_*` verbs — `GAS`/`BRAKE`/`HANDBRAKE`/`STEERLEFT`/`STEERRIGHT`/`NOS`/
`SHIFTUP`/`SHIFTDOWN`/`RESET`/`GAMEBREAKER`; `InputPlayer`/`InputPlayerDrag`/`InputObject`/`InputNIS`; `DirectInput`/
`SteeringWheelDevice`/`Controller`/`ControllerUnplugged`).
