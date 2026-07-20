# C65.5 — Gauges & Meters

> **The one-sentence version:** the gauges and meters are value-bound widgets refreshed every frame in
> `HudElement::Update` — the speedometer from `|velocity|`, the tachometer from RPM, and the busted/get-away bars
> *are* the bust-envelope meter (`[perp+0x120]`) you are literally watching compute.

[← C65.4 — Resolution & widescreen](04-resolution-widescreen.md) · [Chapter 65 hub](C65-HUD-Runtime.md) ·
[Next: C65.6 — Scoreboards →](06-scoreboards.md)

---

## Value bindings: the continuous widgets

The gauges and meters are the HUD's **continuously-updated** widgets — each `HudElement::Update(IPlayer*)`
([C65.2](02-gauge-cluster.md)) pulls a live game value every frame and drives its FEObjects (needle rotation, bar
fill, digits):

| Widget | Bound value | Source |
|---|---|---|
| `Speedometer` | speed = `|velocity|` | `IntegrateMotion` ([C41.5](../C41-Physics-RigidBody/05-integrate-math.md)) |
| `Tachometer` / `Hud_DragTachometer` | engine RPM | engine mechanic ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `Hud_HeatMeter` | pursuit Heat | pursuit ([C48.2](../C48-Pursuit-Heat/02-heat.md)) |
| `pBustedMeter` / `pGetAwayMeter` | the bust envelope meter | `[perp+0x120]` ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)) |
| `Nitrous` / `SpeedBreakerMeter` / `EngineTemp` | NOS / speedbreaker / temp | input + sim |

So the dashboard is a set of **live readouts** — the needle *is* the physics value, refreshed each frame. This is
the tightest sim→HUD coupling ([C65.2](02-gauge-cluster.md)): no game-logic layer, just "read the value, drive the
object."

> ✅ *Verified:* `Speedometer`/`SpeedoUnits`, `Tachometer`, `Hud_DragTachometer`, `Hud_HeatMeter`, `Hud_BustedMeter`,
> `Hud_GetAwayMeter`, `Hud_EngineTempGauge` are present in `speed.exe`. The bust-envelope meter constants are
> byte-verified ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)).

## The gauge cluster: speed & revs

The **speedometer** and **tachometer** (`SpeedoUnits` toggles mph/kph) are the driver's core instruments:

- **Speedometer** — reads `|velocity|` from `IntegrateMotion`'s `Math::Sqrt`
  ([C41.5](../C41-Physics-RigidBody/05-integrate-math.md)); the needle *is* the sim's speed, so it moves exactly as
  you accelerate.
- **Tachometer** — reads engine RPM ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)); the
  shift cue. In drag ([C55.3](../C55-Race-Events/03-drag.md)), the prominent **`Hud_DragTachometer`** marks the
  **perfect-shift window** (`DRAG_MAIN_METER`/`DRAG_REDLINE_*` art, `ENGINE_HEAT_SHIFTLIGHT_GROUP`) — the tach
  becomes the *primary interface* ([C65.6](06-scoreboards.md) covers the drag layout swap).

So the gauge cluster spans **information** (the speedometer, telling you your speed) and **interface** (the drag
tachometer, the thing you *act on* to time shifts). It's the HUD at its most direct — two physics numbers, drawn as
needles.

## The busted bar IS the bust meter

The most instructive binding is the **busted/get-away bars** — they are a *direct display of the byte-verified bust
envelope* ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)):

- **The busted meter** at `[perp+0x120]` fills while you're trapped (in the bust radius, below `BustSpeed`) —
  `dt×0.25` normally, **`dt×4.0` when a cop is engaged** — and drains `dt×0.5` outside.
- **The HUD is notified** via a vtable slot (`+0x3C`) on every **0.1-quantized change** of the gauge — so the bar
  updates as the meter moves.
- **Bustable past 5.0** (`[0x890DA4]`) + a **3.0-second hold** (`[0x8EB318]`) → `MPerpBusted`
  ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)); the mirror is `MPerpEscaped` (the get-away meter).

So when you watch the busted bar fill, **you are literally watching the bust envelope compute**
([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)) — the bar *is* the `[perp+0x120]` gauge, notified at 0.1 steps.
This is the tightest possible HUD-to-mechanic coupling: the meter's *value* is the bar's *fill*. It's why the
busted bar is so tense — a near-full bar means you're seconds from the 3.0s hold completing, and a filling bar with
engaged cops (×4.0 rate) fills alarmingly fast. The pursuit HUD makes the invisible envelope *visible*.

## The garage bars: the same idiom, pure UI

The garage **performance bars** (`TOPSPEED`/`ACCELERATION`/`HANDLING`,
[Chapter 56](../C56-Customization/C56-Customization.md)) show the *same value-binding idiom* in pure-UI form: they
register through a vtable slot (`+0x28`) at fixed owner offsets, range `1..160`, as *display normalizations* —
**not physics** ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)). Editing a bar moves the
*bar*, not the *car*. This confirms the HUD/UI is a pure *readout* ([C65.2](02-gauge-cluster.md)): the bars display
computed values, they don't drive them. The gauge cluster ([above](#the-gauge-cluster-speed--revs)) and the garage
bars are the same pattern — a widget bound to a value — one live (speed), one static (a car's stat). Both *show*,
neither *changes*.

> 🟡 *Reasoned:* the garage-bar mechanism (vtable-slot registration, display normalization) is documented in the
> customization RE ([Chapter 56](../C56-Customization/C56-Customization.md)); it's cited here as the pure-UI form of
> the same value-binding the live gauges use. The live gauge strings and the bust-envelope constants are verified.

## RE implications

- **Gauges/meters are value-bound** — `HudElement::Update` pulls a live value each frame (speed, RPM, heat) and
  drives the widget.
- **Gauge cluster** — speedometer (`|velocity|`) + tachometer (RPM); the drag tach is a *skill interface*.
- **The busted bar IS the bust meter** (`[perp+0x120]`) — notified at 0.1 steps; watching it is watching the
  envelope compute.
- **The garage bars** are the same idiom in pure UI — display normalizations, not physics (editing moves the bar,
  not the car).

---

### Key takeaways

- The gauges/meters are **value-bound** — each `HudElement::Update(IPlayer*)` pulls a **live value every frame**
  and drives its FEObjects (needle/bar/digits).
- The **gauge cluster** reads **speed** (`|velocity|`, [C41.5](../C41-Physics-RigidBody/05-integrate-math.md)) and
  **RPM** ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)); the drag **`Hud_DragTachometer`**
  is elevated to the *primary interface* for shift timing.
- The **busted/get-away bars ARE the bust-envelope meter** (`[perp+0x120]`) — fills `dt×0.25` (×4.0 engaged),
  drains `dt×0.5`, notified to the HUD at **0.1-quantized** changes — you are **watching the envelope compute**.
- The **garage performance bars** are the same value-binding in pure UI — **display normalizations, not physics**
  (editing moves the bar, not the car) — confirming the HUD is a pure **readout**.
- This is the **tightest sim→HUD coupling** — the widget's value *is* the game's value, refreshed each frame.

**Continue:** [C65.6 — Scoreboards](06-scoreboards.md) · [Chapter 65 hub](C65-HUD-Runtime.md)
