# Chapter 53 — Cameras & the Director

> **Goal of this chapter:** decode how Most Wanted frames the action — the gameplay camera (`CameraAI`, with its
> `CAM_ACTION_CHANGE`/`LOOKBACK`/`PULLBACK` verbs, `CameraMover`, `CameraShake`) and the cinematic director (the
> `CDAction*` family — `CDActionDrive`, `CDActionTrackCar`, `CDActionTrackCop`, `CDActionShowcase`) that stages
> cutscenes and showcase shots.

The renderer draws from a *view* ([C51.5](../C51-Render-Pipeline/05-render-frame.md)), and *what that view sees* is
the camera's job. This chapter decodes the two camera systems: the **`CameraAI`** that follows your car during
play (and responds to your look-back/pull-back inputs), and the **cinematic director** (`CDAction*`) that takes
over for cutscenes, Blacklist intros, and beauty shots. Between them they frame everything you see — the everyday
chase camera and the scripted drama.

> **Verified against the executable.** The gameplay camera is **`CameraAI`** (a named singleton system,
> [C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)); its player verbs are `CAM_ACTION_CHANGE`,
> `CAM_ACTION_LOOKBACK`, `CAM_ACTION_PULLBACK` (plus `CAM_GEARS`). Camera behaviours are named: `CameraMover`,
> `CameraShake`, `CameraAnchor`, `CameraFromRacer`, `CameraCutMoment`, `CameraPhotoFinish`, `CameraScreen`,
> `CameraMessagePort`. The **cinematic director** is the `CDAction*` family: `CDActionDrive`, `CDActionTrackCar`,
> `CDActionTrackCop`, `CDActionShowcase`, `CDActionIce`, `CDActionDebug`/`DebugWatchCar` — all present in
> `speed.exe`.

---

## Deep-dive pages

- [C53.1 — The two camera systems](01-two-systems.md): gameplay `CameraAI` vs. cinematic `CDAction*`.
- [C53.2 — CameraAI: the gameplay camera](02-cameraai.md): following the car, the `CAM_ACTION_*` verbs.
- [C53.3 — The cinematic director](03-cinematic-director.md): the `CDAction*` staging of cutscenes.
- [C53.4 — Camera behaviours & moments](04-camera-moments.md): shake, photo-finish, cut moments.
- [C53.5 — Reading cameras in RE](05-reading-cameras.md): navigating the camera systems.

---

## 53.1 Two camera systems

Like effects ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)), cameras come in **two systems**
([C53.1](01-two-systems.md)): the **gameplay camera** (`CameraAI`) that follows your car during play, and the
**cinematic director** (`CDAction*`) that stages scripted shots (cutscenes, showcases,
[Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)). One is *reactive* (following what you do); the other is
*directed* (staging a planned sequence). The game hands control between them — gameplay camera during play, director
during cutscenes.

## 53.2 CameraAI: the gameplay camera

**`CameraAI`** ([C53.2](02-cameraai.md)) is the automatic gameplay camera — it follows your car, choosing framing,
follow distance, and look-ahead so you can see where you're going at speed. It responds to **player verbs**:
`CAM_ACTION_CHANGE` (switch view — bumper/hood/chase), `CAM_ACTION_LOOKBACK` (glance behind — vital for pursuits),
`CAM_ACTION_PULLBACK`. `CameraMover` drives its motion; `CameraShake` adds impact/nitrous shake. It's the camera you
live behind for the whole game.

## 53.3 The cinematic director

The **cinematic director** ([C53.3](03-cinematic-director.md)) is the `CDAction*` family — the *staged* cameras for
cutscenes and showcases: `CDActionDrive` (a driving-follow cinematic), `CDActionTrackCar`/`CDActionTrackCop` (track
a specific car or cop), `CDActionShowcase` (a beauty/orbit shot — the car reveal), `CDActionIce` (a specific
scripted cinematic). These are *director actions* — like the AI actions ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)),
a menu of camera behaviours the director selects to stage a sequence. They drive the NIS cutscenes
([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)).

## 53.4 Camera behaviours & moments

Beyond the two systems are **behaviours and moments** ([C53.4](04-camera-moments.md)) — reusable camera effects and
special shots: `CameraShake` (screen shake on impact/nitrous/pursuit-breaker), `CameraPhotoFinish` (the race-finish
shot), `CameraCutMoment` (a hard cut), `CameraAnchor` (a fixed anchor point), `CameraFromRacer` (a racer-POV view).
These compose into both the gameplay and cinematic cameras, adding the punctuation — the shake of a crash, the
drama of a photo finish — that makes moments *land*.

---

### Key takeaways

- Cameras come in **two systems**: the reactive gameplay camera (**`CameraAI`**) and the directed cinematic camera
  (**`CDAction*`**).
- **`CameraAI`** follows your car and responds to player verbs — `CAM_ACTION_CHANGE` (view toggle),
  `CAM_ACTION_LOOKBACK` (glance back), `CAM_ACTION_PULLBACK` — driven by `CameraMover`.
- The **cinematic director** (`CDActionDrive`/`TrackCar`/`TrackCop`/`Showcase`/`Ice`) stages cutscenes and beauty
  shots — a menu of director actions, like the AI's actions.
- **Camera behaviours & moments** (`CameraShake`, `CameraPhotoFinish`, `CameraCutMoment`) add the punctuation —
  crash shake, race-finish drama.
- The game **hands control** between the gameplay camera (during play) and the director (during cutscenes).

**Next:** [Chapter 54 — GameFlow, Modes & the Blacklist](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md): the
career structure that strings the events together.
