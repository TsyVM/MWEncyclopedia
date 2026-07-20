# C60.4 — InputPlayer

> **The one-sentence version:** `InputPlayer` is the mapper — it reads the active device, applies the control
> binding and response shaping (deadzones, sensitivity), and produces the `GAME_ACTION_*` values for the car's
> INPUT mechanic — with `InputPlayerDrag` the variant for the drag minigame.

[← C60.3 — Devices](03-devices.md) · [Chapter 60 hub](C60-Input-Devices.md) ·
[Next: C60.5 — Reading input in RE →](05-reading-input.md)

---

## The mapper

**`InputPlayer`** is the class that turns *device inputs* ([C60.3](03-devices.md)) into *game actions*
([C60.2](02-game-actions.md)) — the bridge in the input path ([C60.1](01-input-model.md)):

```
InputPlayer, each frame:
   read the active device (DirectInput, C60.3) — its axes and buttons
   apply the control binding — which input maps to which GAME_ACTION_*
   apply response shaping — deadzones, sensitivity, steering speed
   → output the GAME_ACTION_* values (GAS=0.8, STEER=-0.3, ...)
      → the car's INPUT mechanic reads them (C40.3)
```

So `InputPlayer` is where the *device* meets the *game action* — it holds the *mapping* (binding) and the *feel*
(shaping). It's the player's command producer, feeding the `AIVehicleHuman`
([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)) the same way the AI planner feeds an AI car
([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)).

> ✅ *Verified:* `InputPlayer` and `InputPlayerDrag` are present in `speed.exe` — the player input mapper and its
> drag-mode variant. `InputObject` is the base; `InputNIS` handles cutscene input.

## Response shaping: the feel

Beyond raw mapping, `InputPlayer` *shapes* the input — the difference between a device reading and a *good-feeling*
control ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)):

- **Deadzones** — a small stick/wheel movement near centre is ignored (so a resting stick doesn't drift the car) —
  the input only registers past a threshold.
- **Sensitivity curves** — the mapping from device position to game-action value can be *non-linear* — e.g. fine
  control near centre (for small corrections) and faster response at the extremes.
- **Steering speed** — on a *digital* device (keyboard, [C60.3](03-devices.md)), steering must be *smoothed* — a
  key is on/off, but the car needs a gradual steer, so `InputPlayer` ramps `GAME_ACTION_STEER` toward the target
  rather than snapping. This is what makes keyboard steering *usable*.

So `InputPlayer` is where *feel* is tuned — the shaping that makes the controls responsive but not twitchy,
precise but forgiving. This is especially important for *digital* devices ([C60.2](02-game-actions.md)): the
keyboard's binary keys become smooth analog-ish control through `InputPlayer`'s ramping. The response shaping is
the input's version of the mechanics' tuning ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) —
data-driven parameters ([C60.5](05-reading-input.md)) that make each device feel right.

> 🟡 *Reasoned:* the deadzone/sensitivity/steering-speed shaping is the standard input-processing model, consistent
> with `InputPlayer`'s role and the analog/digital device split ([C60.3](03-devices.md)); the exact shaping curves
> are per-config data. `InputPlayer`/`InputPlayerDrag` are verified.

## InputPlayerDrag: the mode variant

The verified **`InputPlayerDrag`** is a *variant* of `InputPlayer` for the **drag minigame**
([C55.3](../C55-Race-Events/03-drag.md)) — because drag racing needs a *different* control scheme:

- **Steering is automated** ([C55.3](../C55-Race-Events/03-drag.md)) — the car holds its lane, so `InputPlayerDrag`
  doesn't map full steering (or maps only lane-changes).
- **The focus is shift timing** — the perfect-shift window ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md))
  is the key skill, so `InputPlayerDrag` emphasises the `SHIFTUP`/`SHIFTDOWN` and launch actions.

So the game *swaps the input mapper* for the drag mode — `InputPlayerDrag` instead of `InputPlayer` — giving drag
racing its distinct controls ([C55.3](../C55-Race-Events/03-drag.md)) while reusing the same devices
([C60.3](03-devices.md)) and game actions ([C60.2](02-game-actions.md)). This is the *mode-swaps-a-component*
pattern again ([C55.3](../C55-Race-Events/03-drag.md), where drag swaps `EngineDragster`): the drag mode swaps the
engine mechanic *and* the input mapper, giving it a wholly different feel from the same parts. That there's a
*named* drag input class confirms the depth of the mode-specific customization — drag isn't just different rules,
it's a different *control scheme*, coded as `InputPlayerDrag`.

## InputPlayer completes the driver

With `InputPlayer` decoded, the *player driver* is complete ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)):

- **The device** ([C60.3](03-devices.md)) provides the raw input.
- **`InputPlayer`** maps and shapes it into game actions ([C60.2](02-game-actions.md)).
- **The `AIVehicleHuman`** ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)) reads the actions via its INPUT
  mechanic ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)).
- **The driving model** ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md))
  turns them into motion.

So `InputPlayer` is the piece that was implicit in the driver chapters ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md))
— "the player's car takes commands from `InputPlayer`" ([C47.2](../C47-AI-Driver-Vehicle/02-player-is-ai.md)) — now
decoded. The player and AI ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) are fully symmetric
command producers, and `InputPlayer` is the player's. Reading it closes the driver loop: from the hand on the
device to the car on the road.

## RE implications

- **`InputPlayer`** maps device inputs to game actions — the binding + response shaping, feeding the INPUT mechanic.
- **Response shaping** (deadzones, sensitivity, steering speed) tunes the *feel* — crucial for digital devices
  (keyboard smoothing).
- **`InputPlayerDrag`** is the drag-mode variant — automated steering, shift focus — the mode-swaps-a-component
  pattern.
- **`InputPlayer` completes the player driver** — device → map/shape → game actions → `AIVehicleHuman` → motion.

---

### Key takeaways

- **`InputPlayer`** is the **mapper** — it reads the active device, applies the **control binding** and **response
  shaping** (deadzones, sensitivity, steering speed), and outputs `GAME_ACTION_*` values for the INPUT mechanic.
- **Response shaping** tunes the **feel** — especially smoothing **digital** devices (a keyboard's binary keys
  become gradual steer via ramping), the input's version of data-driven tuning.
- **`InputPlayerDrag`** is the **drag-mode variant** — automated steering, shift-timing focus — the
  *mode-swaps-a-component* pattern (drag also swaps `EngineDragster`,
  [C55.3](../C55-Race-Events/03-drag.md)).
- `InputPlayer` **completes the player driver** — the piece implicit in Chapter 47 ("the player's car takes
  commands from `InputPlayer`"), now decoded.
- The player and AI are **fully symmetric command producers** — `InputPlayer` (device) and the goal/action planner
  (plan) — converging at the `GAME_ACTION_*` interface.

**Continue:** [C60.5 — Reading input in RE](05-reading-input.md) · [Chapter 60 hub](C60-Input-Devices.md)
