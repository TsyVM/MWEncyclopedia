# C53.1 — The Two Camera Systems

> **The one-sentence version:** cameras split into two systems — the reactive gameplay camera (`CameraAI`, which
> follows your car and obeys your look-back/view inputs) and the directed cinematic camera (`CDAction*`, which
> stages planned cutscene and showcase shots) — with control handed between them by game state.

[← Chapter 53 hub](C53-Cameras-Director.md) · [Next: C53.2 — CameraAI →](02-cameraai.md)

---

## Reactive vs. directed

The camera answers "what does the view see?" ([C51.5](../C51-Render-Pipeline/05-render-frame.md)), and MW answers
it two ways depending on context:

- **Reactive (`CameraAI`).** During play, the camera *follows what you do* — it tracks your car, frames the road
  ahead, and reacts to your inputs (look back, change view). It doesn't know the future; it responds to the present
  ([C53.2](02-cameraai.md)).
- **Directed (`CDAction*`).** During a cutscene, showcase, or Blacklist intro, the camera *stages a planned shot* —
  a director ([C53.3](03-cinematic-director.md)) chooses framing, movement, and cuts to tell a scripted moment. It
  knows the sequence and composes for it.

These are different jobs: one *chases* the action, the other *presents* it. A racing game needs both — a
functional camera to *play* through and a cinematic camera to *watch* the drama (your car reveal, a rival's intro).

> ✅ *Verified:* the gameplay camera is `CameraAI` (a named system, [C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md));
> the cinematic director is the `CDAction*` family (`CDActionDrive`/`TrackCar`/`TrackCop`/`Showcase`/`Ice`) — both
> present in `speed.exe`.

## Handing over control

The two systems don't run at once — the game **hands control** between them by state:

- **During gameplay** (driving, racing, pursuit) → `CameraAI` ([C53.2](02-cameraai.md)) drives the view.
- **During a cutscene / showcase / NIS** ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)) → the cinematic
  director ([C53.3](03-cinematic-director.md)) takes over.
- **At transitions** (a race finish, a bust) → a *moment* camera ([C53.4](04-camera-moments.md), like
  `CameraPhotoFinish`) may bridge.

So the view is always owned by *one* camera authority, swapped at the seams between play and cinematic. This is why
the transition from driving into a cutscene feels clean — control passes from `CameraAI` to the director, the
director stages its shot, and control returns. The GameFlow phase ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md))
often drives the handover (entering a cutscene phase gives the director the camera).

## The parallel with effects and AI

The two-camera split mirrors patterns seen elsewhere ([C52.1](../C52-Effects-Particles/01-two-worlds.md)):

- **Two effect worlds** ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)) — scene particles vs.
  screen post-process.
- **Two driver sources** ([C40.3](../C40-Eight-Mechanics/03-ai-and-input.md)) — player input vs. AI planner.
- **Two camera systems** (this chapter) — reactive gameplay vs. directed cinematic.

The recurring shape is a *reactive/present* mode and a *directed/planned* mode of the same faculty. And the
cinematic director's structure — a family of `CDAction*` behaviours a director selects
([C53.3](03-cinematic-director.md)) — directly parallels the AI's `AIAction*` menu
([Chapter 46](../C46-AI-Goals-Actions/05-action-menu.md)): the same "director selects from a menu of actions"
architecture, applied to cameras instead of driving. Recognising this parallel makes the camera director immediately
legible if you've read the AI chapters — it's the same design, staging shots instead of driving cars.

## Why two systems

Splitting cameras into gameplay and cinematic is a clean separation of concerns:

- **Gameplay needs *function*.** The play camera must always show you what you need to drive — the road, the
  threats — reacting instantly to your speed and inputs ([C53.2](02-cameraai.md)). It's optimised for *legibility*.
- **Cinematics need *composition*.** The director camera must frame for *drama* — angles, movement, cuts — planned
  for effect ([C53.3](03-cinematic-director.md)). It's optimised for *presentation*.

One camera can't do both well — a cinematic camera would be disorienting to drive with, and a functional camera
would be flat for cutscenes. So MW builds two, each specialised, and swaps them by context. This is the same
engineering judgement as the rest of the engine: build the right tool for each job, and compose them at the seams.
The result is a game that both *plays* well (the `CameraAI`) and *presents* well (the director) — cameras that know
which mode the moment calls for.

## RE implications

- **Two camera systems** — reactive gameplay (`CameraAI`) and directed cinematic (`CDAction*`).
- **Control is handed** between them by game state — one authority owns the view at a time.
- **The parallel** — reactive/directed mirrors effects (particles/post-process) and AI (input/planner); the
  director's `CDAction*` menu mirrors the AI's `AIAction*`.
- **Two systems** because gameplay needs *function* (legibility) and cinematics need *composition* (drama).

---

### Key takeaways

- Cameras split into **two systems**: the **reactive gameplay camera** (`CameraAI`, follows your car and inputs)
  and the **directed cinematic camera** (`CDAction*`, stages planned shots).
- The game **hands control** between them by state — gameplay camera during play, director during cutscenes — so
  one authority owns the view at a time (clean transitions).
- The split **parallels** the two effect worlds and the two driver sources; the director's `CDAction*` menu mirrors
  the AI's `AIAction*` architecture.
- Two systems exist because gameplay needs **function** (show what you need to drive) and cinematics need
  **composition** (frame for drama) — one camera can't do both well.
- The result **plays well and presents well** — the right camera for each moment.

**Continue:** [C53.2 — CameraAI: the gameplay camera](02-cameraai.md) · [Chapter 53 hub](C53-Cameras-Director.md)
