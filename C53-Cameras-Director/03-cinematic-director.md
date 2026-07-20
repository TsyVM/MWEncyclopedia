# C53.3 — The Cinematic Director

> **The one-sentence version:** the cinematic director stages scripted shots through the `CDAction*` family — a
> menu of camera behaviours (`CDActionDrive`, `CDActionTrackCar`, `CDActionTrackCop`, `CDActionShowcase`,
> `CDActionIce`) the director selects to compose cutscenes, Blacklist intros, and car reveals — the same
> menu-of-actions design as the AI.

[← C53.2 — CameraAI](02-cameraai.md) · [Chapter 53 hub](C53-Cameras-Director.md) ·
[Next: C53.4 — Camera behaviours & moments →](04-camera-moments.md)

---

## The director actions

When the game needs a *composed* shot ([C53.1](01-two-systems.md)) — a cutscene, a showcase, a rival's intro — the
**cinematic director** takes over, staging the camera through the **`CDAction*`** family (CD = Cinematic Director).
Each `CDAction*` is a *camera behaviour* the director can run:

| Director action | The shot it stages |
|---|---|
| `CDActionDrive` | a driving-follow cinematic — the car driving, framed dramatically |
| `CDActionTrackCar` | track a specific car — hold it in frame as it moves |
| `CDActionTrackCop` | track a cop — the pursuit-cinematic view of a cruiser |
| `CDActionShowcase` | a showcase/orbit — the beauty shot (car reveal, garage) |
| `CDActionIce` | a specific scripted cinematic ("Ice" — a set-piece) |
| `CDActionDebug` / `DebugWatchCar` | development camera actions |

So the director has a *repertoire* of shot types, and it composes a sequence by choosing among them — track this
car, cut to a showcase, follow the drive. This is *directed* camera work: planned, composed, for effect.

> ✅ *Verified:* the `CDAction*` family — `CDActionDrive`, `CDActionTrackCar`, `CDActionTrackCop`, `CDActionShowcase`,
> `CDActionIce`, `CDActionDebug`, `CDActionDebugWatchCar` — are present as strings in `speed.exe`. `CameraFromRacer`
> and `CameraAnchor` ([C53.4](04-camera-moments.md)) support director framing.

## The same design as the AI

The cinematic director's structure is *directly parallel* to the AI's goal/action system
([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) — a striking reuse of architecture:

- **The AI** has `AIAction*` behaviours ([Chapter 46](../C46-AI-Goals-Actions/05-action-menu.md)) — Race, Ram,
  GetUnstuck — selected each tick to drive a car.
- **The director** has `CDAction*` behaviours (this page) — Drive, TrackCar, Showcase — selected to stage a shot.

Both are a *director selecting from a menu of actions*. The AI director (a goal) picks driving actions; the
cinematic director picks camera actions. The `CDAction*` naming even mirrors `AIAction*`. This is the engine reusing
its **action-selection pattern** ([C46.1](../C46-AI-Goals-Actions/01-goals-and-actions.md)) for a completely
different domain — cameras instead of driving. So if you understand the AI's action menu
([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), you understand the cinematic director: it's the
same machine, staging cameras. This kind of pattern reuse across domains is a hallmark of a mature engine — one good
idea (menu-of-actions) applied wherever a "select the right behaviour" problem appears.

> 🟡 *Reasoned:* the director-selects-CDAction parallel to the AI's goal/action model is inferred from the identical
> `*Action*` naming and role (a menu of behaviours a director chooses); the exact director implementation is deeper
> RE. The `CDAction*` family is verified.

## Staging a cutscene

A cutscene ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)) is the director running a *sequence* of
`CDAction*` shots, synchronised to the scripted animation ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)):

```
NIS cutscene (Ch 24-25):
   the NIS timeline plays the scene animation
   the director stages the camera:
      CDActionTrackCar  — follow the hero car in
      CDActionShowcase  — orbit for the reveal
      CDActionTrackCop  — cut to the pursuing cruiser
      ...timed to the NIS event track (Ch 25)
```

So the director is the *camera track* of a cutscene — while the NIS system animates the cars and characters
([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)), the director frames them with a sequence of shots, cut
and timed to the moment ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)). This is how MW's cutscenes (the
Blacklist rival intros, the story beats) are staged — the director choosing `CDAction*` shots to present the
scripted action cinematically. `CDActionShowcase` in particular is the *car reveal* shot — the slow orbit of a new
car that's a staple of the genre.

## Why a director, not fixed cameras

Using a director that *selects* shots (rather than pre-baked fixed camera paths) is flexible and reusable:

- **Reusable shot types.** `CDActionTrackCar` works for *any* car in *any* cutscene — the director points it at the
  relevant car. One shot behaviour, many uses (like the AI's reusable actions,
  [C46.5](../C46-AI-Goals-Actions/05-action-menu.md)).
- **Composable sequences.** A cutscene is *composed* from the shot menu — mix and cut `CDAction*` shots to stage any
  scene, without authoring bespoke camera code per cutscene.
- **Consistent with the engine.** The menu-of-actions pattern ([above](#the-same-design-as-the-ai)) is used
  throughout, so the director fits the engine's architecture — same machinery, new domain.

So the cinematic director is the *composition* engine for cameras: a reusable menu of shot behaviours, selected and
sequenced to stage any cutscene or showcase. It's the counterpart to `CameraAI` ([C53.2](02-cameraai.md)) — where
the gameplay camera *reacts*, the director *composes*, and both draw on shared camera behaviours
([C53.4](04-camera-moments.md)). Together they frame the whole game, function and drama.

## RE implications

- **The cinematic director** stages shots via the `CDAction*` family — Drive, TrackCar, TrackCop, Showcase, Ice.
- **Same design as the AI** — a director selecting from a menu of `*Action*` behaviours
  ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) — pattern reuse across domains.
- **Cutscenes** are director sequences of `CDAction*` shots, timed to the NIS track
  ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)).
- **A director, not fixed cameras** — reusable shot types, composable sequences, engine-consistent.

---

### Key takeaways

- The **cinematic director** stages scripted shots through the **`CDAction*`** family — `CDActionDrive`,
  `CDActionTrackCar`, `CDActionTrackCop`, `CDActionShowcase` (the car reveal), `CDActionIce`.
- Its design is **directly parallel to the AI's goal/action system** — a director selecting from a **menu of
  `*Action*` behaviours** — the engine reusing its action-selection pattern for cameras.
- A **cutscene** is the director running a *sequence* of `CDAction*` shots, cut and timed to the NIS animation track
  ([Chapters 24](../C24-NIS-Animation/C24-NIS-Animation.md)–[25](../C25-NIS-Events/C25-NIS-Events.md)).
- Using a **director that selects shots** (not fixed camera paths) gives reusable shot types and composable
  sequences — author a cutscene from the menu, no bespoke camera code.
- The director **composes** where `CameraAI` **reacts** — together they frame the whole game, drama and function.

**Continue:** [C53.4 — Camera behaviours & moments](04-camera-moments.md) · [Chapter 53 hub](C53-Cameras-Director.md)
