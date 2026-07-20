# C60.3 — Devices

> **The one-sentence version:** physical devices are accessed via DirectInput — `SteeringWheelDevice` (wheel with
> pedals/force feedback), a `Controller` (gamepad), and the keyboard — each presenting axes/buttons that map to
> the game actions, with `ControllerUnplugged` handling disconnection.

[← C60.2 — The game actions](02-game-actions.md) · [Chapter 60 hub](C60-Input-Devices.md) ·
[Next: C60.4 — InputPlayer →](04-inputplayer.md)

---

## DirectInput: the device API

Most Wanted accesses input devices through **`DirectInput`** — the DirectX 8/9-era input API (contemporary with
its Direct3D 9 renderer, [C51.1](../C51-Render-Pipeline/01-d3d9-foundation.md)). DirectInput enumerates and reads
the connected devices — gamepads, wheels, joysticks, and the keyboard — presenting each as a set of *axes* (analog)
and *buttons* (digital). So the input system's foundation is DirectInput, just as the renderer's is Direct3D 9
([C51.1](../C51-Render-Pipeline/01-d3d9-foundation.md)) — the standard 2005 Windows-game APIs.

> ✅ *Verified:* `DirectInput` is referenced in `speed.exe`, alongside `Device`, `SteeringWheelDevice`,
> `Controller`, and `ControllerUnplugged` — the device layer.

## The device types

The verified device classes cover the three input types a 2005 racer supports:

- **`SteeringWheelDevice`** — a **racing wheel**: an analog steering axis, analog pedals (accelerator/brake), and
  buttons — plus (typically) **force feedback**. This is the *precision* device — fine analog `GAS`/`STEER`
  ([C60.2](02-game-actions.md)) and the physical feedback of a wheel.
- **`Controller`** — a **gamepad**: analog sticks and triggers, plus buttons. Analog throttle/steer via the
  triggers/stick, digital actions via buttons — the *common* device.
- **The keyboard** — keys only (digital) — full-on/full-off `GAS`/`STEER` ([C60.2](02-game-actions.md)), a *binary*
  approximation of the analog actions.

So the game supports the full spectrum — wheel (best precision), pad (good), keyboard (functional). Each is a
device presenting inputs that `InputPlayer` ([C60.4](04-inputplayer.md)) maps to the shared game actions. The
`SteeringWheelDevice` being a *named class* (vs. a generic controller) reflects the wheel's special features
(pedals, force feedback) needing dedicated handling.

## Force feedback: the wheel's output

The `SteeringWheelDevice` is notable for being *bidirectional* — it's not just an *input* but an *output* too, via
**force feedback**:

- **The wheel pushes back** — the game sends forces to the wheel to simulate the car's feel: the resistance of
  cornering, the kick of a curb, the shudder of a collision ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)),
  the loss of grip in a slide ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)).
- **Driven by the sim** — the force-feedback signal comes from the car's physics
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) — the tyre forces, the
  impacts — so the wheel *feels* what the car feels.

So the wheel closes a *physical loop*: the player turns the wheel (input → `GAME_ACTION_STEER`), the car responds
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)), and the car's forces push
back through the wheel (force feedback → the player's hands). This is the most *immersive* input — you *feel* the
road through the wheel, driven by the same sim state ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
as the visuals and audio. Force feedback is thus another *presentation channel* ([C53.4](../C53-Cameras-Director/04-camera-moments.md))
— alongside the screen, the speakers, and the camera shake, the wheel conveys the car's state to the player,
haptically.

> 🟡 *Reasoned:* the force-feedback-from-sim-forces model is the standard racing-wheel design, consistent with the
> verified `SteeringWheelDevice` class and DirectInput's force-feedback support; the exact FFB signal computation
> is deeper RE. The device classes and DirectInput are verified.

## ControllerUnplugged: robustness

The verified **`ControllerUnplugged`** handles a device being *disconnected* mid-play — a real concern (a wireless
pad dies, a cable is pulled). Handling it explicitly:

- **Pauses/prompts** — the game detects the loss and typically pauses, prompting the player to reconnect
  ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) — avoiding an uncontrolled car.
- **Re-enumerates** — on reconnection, the device is re-detected and input resumes.

That this is a *named* case reflects the input system's *robustness* — a shipped game must handle devices coming and
going gracefully. It's a small but important detail of a production input system: the happy path (read the device)
plus the failure path (device unplugged). Reading `ControllerUnplugged` confirms the input system was built for the
real world of flaky hardware, not just the ideal case — the same production maturity seen in the streaming
system's blocking loads ([C38.6](../C38-Resource-Streaming-Residency/06-blocking-budgets.md)) and the spawn retries
([C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)).

## RE implications

- **`DirectInput`** is the device API — enumerating pads, wheels, and the keyboard as axes/buttons.
- **Device types** — `SteeringWheelDevice` (wheel, precision + FFB), `Controller` (pad), keyboard (digital).
- **Force feedback** — the wheel is bidirectional; the car's sim forces push back through it (a haptic
  presentation channel).
- **`ControllerUnplugged`** handles disconnection — the input system's production robustness.

---

### Key takeaways

- Devices are accessed via **`DirectInput`** (the DX-era input API, contemporary with the D3D9 renderer) —
  presenting each as **axes** (analog) and **buttons** (digital).
- The device types span the spectrum: **`SteeringWheelDevice`** (wheel — precise analog + force feedback),
  **`Controller`** (gamepad), and the **keyboard** (digital, binary approximations).
- **Force feedback** makes the wheel **bidirectional** — the car's sim forces (cornering, curbs, impacts, grip
  loss) push back through it, a **haptic presentation channel** driven by the same physics as the visuals/audio.
- **`ControllerUnplugged`** handles mid-play disconnection (pause/prompt/re-enumerate) — the input system's
  **production robustness**.
- Each device maps its inputs to the **shared game actions** ([C60.2](02-game-actions.md)) — the wheel drives best
  (fine analog), the keyboard functionally (binary).

**Continue:** [C60.4 — InputPlayer](04-inputplayer.md) · [Chapter 60 hub](C60-Input-Devices.md)
