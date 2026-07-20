# C60.1 — The Input Model

> **The one-sentence version:** the input path is device → abstract `GAME_ACTION_*` verb → the car's INPUT
> mechanic → controls, so the car reads *actions* (GAS, STEER) regardless of which device produced them.

[← Chapter 60 hub](C60-Input-Devices.md) · [Next: C60.2 — The game actions →](02-game-actions.md)

---

## Abstraction: actions, not devices

The input system's central idea is **abstraction** — the car doesn't read a *device*; it reads *abstract actions*:

```
physical device (pad trigger / wheel pedal / keyboard key)
   → InputPlayer maps it → GAME_ACTION_GAS (a value 0..1)
      → the car's INPUT mechanic reads GAME_ACTION_GAS as throttle (C40.3)
         → the engine mechanic uses the throttle (C42.2)
```

So the throttle can come from *any* device — a gamepad's right trigger, a wheel's accelerator pedal, or a keyboard
key — and the car sees the *same* `GAME_ACTION_GAS` value. The device is abstracted away behind the game action.
This is the input counterpart of the render/audio abstraction ([C51.2](../C51-Render-Pipeline/02-render-objects.md),
[C59.1](../C59-Audio-Runtime/01-audio-runtime.md)) — a layer that decouples the specifics (the device) from the
consumer (the car).

> ✅ *Verified:* the abstraction is named — `GAME_ACTION_*` verbs ([C60.2](02-game-actions.md)), `InputPlayer` (the
> mapper), and `DirectInput` devices ([C60.3](03-devices.md)) — present in `speed.exe`.

## Why abstract the input

Abstracting device inputs into `GAME_ACTION_*` verbs ([above](#abstraction-actions-not-devices)) is essential for
a game supporting many devices:

- **Device independence.** The car ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) and the whole game logic
  work in *game actions*, never in device specifics. Add a new device type (a new wheel), and only the *mapping*
  ([C60.4](04-inputplayer.md)) changes — the game logic is untouched.
- **Remappable controls.** Because the mapping (device input → game action) is *data* ([C60.4](04-inputplayer.md)),
  the player can *remap* controls — assign NOS to a different button — without changing the game. The action is
  fixed; the binding is configurable.
- **Consistent behaviour.** `GAME_ACTION_GAS` behaves identically regardless of source, so the car drives the same
  on a pad or a wheel (modulo the device's precision) — the game feel is device-independent at the logic level.

So the game-action abstraction is what lets Most Wanted support gamepads, wheels, and keyboards
([C60.3](03-devices.md)) with one game logic — a clean separation of *input source* from *input meaning*. The
`GAME_ACTION_*` verbs ([C60.2](02-game-actions.md)) are the stable *meaning*; the devices and mapping are the
variable *source*.

## The player mirrors the AI

The input model exactly mirrors the AI's command production ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) —
both feed the *same* car:

- **The player** produces game actions from a *device* (via `InputPlayer`, [C60.4](04-inputplayer.md)).
- **The AI** produces the equivalent commands from a *plan* (via the goal/action system,
  [Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)).
- **Both** feed the car's driving — the player via the INPUT mechanic, the AI via the AI mechanic
  ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)).

This is why the player's car is an `AIVehicleHuman` ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)) — the
*same* brain wiring as an AI car, just commanded by `InputPlayer` instead of a planner. The `GAME_ACTION_*`
abstraction is the shared *command interface* — the player's device inputs and the AI's decisions both become car
commands, converging at the driving model ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
So the input model completes the driver picture ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)): player and
AI, two sources, one command interface, one car.

## The full command path

Putting it together, the player's command path is:

```
device (pad/wheel/keyboard, via DirectInput, C60.3)
   → InputPlayer: map + shape → GAME_ACTION_* values (C60.2, C60.4)
      → AIVehicleHuman's INPUT mechanic reads the actions (C40.3, C47.2)
         → the driving model: engine, suspension, tyres (Ch 42)
            → the rigid body integrates → the car moves (Ch 39)
```

So a trigger-pull travels: device → `GAME_ACTION_GAS` → throttle → engine torque → wheel force → motion. Every step
is decoded elsewhere in the book; this chapter is the *first* step — turning a physical input into a game action
the car can read. The abstraction ([above](#abstraction-actions-not-devices)) is the hinge: it's where the
*physical* (a device) becomes the *logical* (a game action), and everything downstream is device-independent.

## RE implications

- **Input is abstracted** — device → `GAME_ACTION_*` verb → the car's INPUT mechanic — the car reads actions, not
  devices.
- **The abstraction buys** device independence, remappable controls, and consistent behaviour.
- **The player mirrors the AI** — both produce car commands (device vs. plan), converging at the `GAME_ACTION_*`
  interface.
- **The full path** — device → `InputPlayer` → game actions → INPUT mechanic → driving model → motion.

---

### Key takeaways

- The input path is **device → abstract `GAME_ACTION_*` verb → the car's INPUT mechanic → controls** — the car
  reads *actions* (GAS, STEER), never devices.
- The **abstraction** buys **device independence** (add a device, only the mapping changes), **remappable
  controls** (the binding is data), and **consistent behaviour** (an action behaves the same from any source).
- The player model **mirrors the AI** — both produce car commands (from a device vs. a plan), converging at the
  `GAME_ACTION_*` command interface — which is why the player's car is an `AIVehicleHuman`
  ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)).
- The **full command path** runs device → `InputPlayer` → game actions → driving model → motion — this chapter is
  the **first step** (physical input → game action).
- The abstraction is the **hinge** where the physical becomes the logical, making everything downstream
  device-independent.

**Continue:** [C60.2 — The game actions](02-game-actions.md) · [Chapter 60 hub](C60-Input-Devices.md)
