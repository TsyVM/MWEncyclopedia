# C53.2 — CameraAI: the Gameplay Camera

> **The one-sentence version:** `CameraAI` is the automatic gameplay camera — it follows your car (framing,
> distance, look-ahead) via `CameraMover`, and responds to player verbs `CAM_ACTION_CHANGE` (switch view),
> `CAM_ACTION_LOOKBACK` (glance behind), and `CAM_ACTION_PULLBACK`.

[← C53.1 — The two camera systems](01-two-systems.md) · [Chapter 53 hub](C53-Cameras-Director.md) ·
[Next: C53.3 — The cinematic director →](03-cinematic-director.md)

---

## Following the car

`CameraAI` ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)) is the camera you play behind — it
automatically follows your car so you can drive at speed. Its job each frame is to choose:

- **Framing** — where the camera sits relative to the car (behind and above, for the chase view).
- **Follow distance** — how far back; it pulls back at speed (more field of view, more warning) and closes in when
  slow.
- **Look-ahead** — leading the car into turns, so you see where you're going, not just where you are.
- **Smoothing** — the camera lags and eases rather than rigidly locking, so fast motion isn't jarring.

`CameraMover` (verified) is the component that drives this motion — moving the camera smoothly toward its target
framing each frame. So `CameraAI` is a *reactive* system ([C53.1](01-two-systems.md)): it reads the car's state
(position, speed, heading from the sim, [Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) and
continuously repositions to keep a good driving view.

> ✅ *Verified:* `CameraAI` and `CameraMover` are present in `speed.exe`; the player camera verbs
> `CAM_ACTION_CHANGE`, `CAM_ACTION_LOOKBACK`, `CAM_ACTION_PULLBACK` (and `CAM_GEARS`) are verified strings.

## The player verbs

`CameraAI` responds to **player camera inputs** — verified `CAM_ACTION_*` verbs
([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md) uses the same verb pattern for game actions):

- **`CAM_ACTION_CHANGE`** — switch the camera view. MW offers several (bumper, hood, close chase, far chase), and
  this verb cycles them. Each is a different framing of the same follow behaviour.
- **`CAM_ACTION_LOOKBACK`** — glance behind. *Vital in a pursuit* — you look back to see the cops closing, judge a
  ram, or check a roadblock behind you ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).
  The camera swings to look rearward while held.
- **`CAM_ACTION_PULLBACK`** — pull the camera back for a wider view.
- **`CAM_GEARS`** — a gear-related camera cue (likely the drag-race gear-shift framing,
  [C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)).

So the player has *some* control over the reactive camera — not full manual control, but view selection and
tactical glances. `CAM_ACTION_LOOKBACK` especially ties the camera to gameplay: seeing behind you is a pursuit
skill, and the camera supports it as a verb. These verbs are the player's interface to `CameraAI`, layered over its
automatic following.

## Reading the car's state

Because `CameraAI` follows the car, it *reads the sim* ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
— it's another connector-fed reader ([C52.5](../C52-Effects-Particles/05-reading-effects.md)) of the physics:

- **Position/orientation** ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)) — to place and aim the camera.
- **Speed** ([C41.5](../C41-Physics-RigidBody/05-integrate-math.md)) — to set follow distance and FOV (faster = pulled
  back, wider — the *sense of speed*).
- **Heading/turn** — to lead into corners (look-ahead).

So the camera's feel is *driven by the physics*: the pull-back and FOV widening at high speed (which makes fast
driving feel fast) come from `CameraAI` reading the car's speed. This is a subtle but crucial contribution to game
feel — the camera *amplifies* the sensation of velocity by reacting to it. Like the visual treatment's speed blur
([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)), the camera is part of *selling* speed, driven by the same
sim state.

> 🟡 *Reasoned:* the speed-driven follow-distance/FOV and corner look-ahead are the standard chase-camera design,
> consistent with `CameraAI`/`CameraMover` reading the car's sim state; the exact camera parameters are per-config
> data. The camera classes and player verbs are verified.

## Why an automatic camera

Making the gameplay camera *automatic* (rather than manual) is essential for a driving game:

- **You can't drive and aim a camera.** At racing speed, manually controlling the camera is impossible — the camera
  must follow automatically so you can focus on driving.
- **It optimises for legibility.** `CameraAI` frames for *seeing the road and threats* — the functional priority
  ([C53.1](01-two-systems.md)) — better than a human could while racing.
- **It gives limited, useful control.** The verbs ([above](#the-player-verbs)) let you adjust *when it matters*
  (view choice, look back) without the burden of full control.

So `CameraAI` is the pragmatic gameplay-camera solution: automatic following for the 99% case, with player verbs for
the tactical moments. It's the camera that makes Most Wanted *playable* at speed — reading the physics, following
smoothly, and handing you a look-back when the cops are on your tail. The cinematic director
([C53.3](03-cinematic-director.md)) handles the moments when *composition* matters more than *function*.

## RE implications

- **`CameraAI`** is the automatic gameplay camera — follows the car (framing, distance, look-ahead) via
  `CameraMover`.
- **Player verbs** — `CAM_ACTION_CHANGE` (view), `CAM_ACTION_LOOKBACK` (glance back), `CAM_ACTION_PULLBACK` —
  limited control over the reactive camera.
- **It reads the sim** — position, speed, heading — driving the speed-sense (pull-back/FOV at speed).
- **Automatic** because you can't drive and aim; it optimises for legibility with tactical verbs.

---

### Key takeaways

- **`CameraAI`** is the **automatic gameplay camera** — it follows your car (framing, follow distance, corner
  look-ahead, smoothing) via **`CameraMover`**.
- It responds to **player verbs**: `CAM_ACTION_CHANGE` (cycle view), `CAM_ACTION_LOOKBACK` (glance behind — vital in
  pursuits), `CAM_ACTION_PULLBACK` (wider view).
- It **reads the car's sim state** (position, **speed**, heading) — the speed-driven pull-back and FOV widening
  **amplify the sense of velocity** (like the visual treatment's speed blur).
- **Automatic following** is essential — you can't drive and aim at speed; the camera optimises for **legibility**
  with tactical verbs for the moments that matter.
- It's the camera that makes MW **playable** — the cinematic director handles the moments needing **composition**.

**Continue:** [C53.3 — The cinematic director](03-cinematic-director.md) · [Chapter 53 hub](C53-Cameras-Director.md)
