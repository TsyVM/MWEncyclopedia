# C53.4 — Camera Behaviours & Moments

> **The one-sentence version:** reusable camera behaviours and special-moment shots — `CameraShake` (impact/nitrous/
> pursuit-breaker shake), `CameraPhotoFinish` (the race-finish shot), `CameraCutMoment` (a hard cut), `CameraAnchor`
> (a fixed point), `CameraFromRacer` (a racer POV) — compose into both the gameplay and cinematic cameras, adding
> the punctuation that makes moments land.

[← C53.3 — The cinematic director](03-cinematic-director.md) · [Chapter 53 hub](C53-Cameras-Director.md) ·
[Next: C53.5 — Reading cameras in RE →](05-reading-cameras.md)

---

## Shared camera behaviours

Beyond the two systems ([C53.1](01-two-systems.md)) are **reusable camera behaviours** — effects and special shots
that *both* the gameplay camera and the director draw on. The verified set:

| Behaviour | What it does |
|---|---|
| `CameraShake` | screen shake — on impact, nitrous, pursuit-breaker |
| `CameraPhotoFinish` | the dramatic race-finish shot |
| `CameraCutMoment` | a hard cut (instant camera change) |
| `CameraAnchor` | a fixed anchor point the camera can hold |
| `CameraFromRacer` | a view from a racer's perspective |
| `CameraFinished` | end-of-sequence camera state |
| `CameraScreen` | a screen-space camera element |

These aren't a *system* on their own — they're *behaviours* composed into the gameplay camera
([C53.2](02-cameraai.md)) and the director's shots ([C53.3](03-cinematic-director.md)). `CameraShake` punctuates a
crash whether you're driving (gameplay camera) or watching (director); `CameraPhotoFinish` stages a race ending.

> ✅ *Verified:* `CameraShake`, `CameraPhotoFinish`, `CameraCutMoment`, `CameraAnchor`, `CameraFromRacer`,
> `CameraFinished`, and `CameraScreen` are present as strings in `speed.exe` — the shared camera behaviours.

## CameraShake: punctuating impact

**`CameraShake`** is the most-felt behaviour — the screen shake that punctuates violent moments:

- **On collision** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) — a crash shakes the camera,
  selling the impact's force. A bigger hit shakes harder.
- **On nitrous** ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) — the surge of NOS rattles
  the view, adding to the sense of raw acceleration.
- **On a pursuit breaker** ([C49.5](../C49-Cops-Dispatch-Roadblocks/05-spikes-breakers.md)) — the environment
  crashing down shakes the camera, making the moment visceral.

So `CameraShake` is the camera's contribution to *feedback* — it makes forces *felt* through the view. It reads the
same events as the effects ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)) and sound
([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) — a collision produces sparks (effect), a crunch
(sound), *and* a shake (camera), all reading the one event. This multi-channel feedback (visual + audio + camera) is
what makes an impact *land*: three systems reacting to the same collision. `CameraShake` is the camera's channel.

## CameraPhotoFinish and moments

**`CameraPhotoFinish`** is a *moment camera* — a special shot for a specific dramatic beat (a race's finish):

- **The photo finish** — as you cross the line, the camera cuts to a dramatic angle (a low, sweeping shot of the
  finish) to celebrate the win. It's a scripted *moment* bridging gameplay and result
  ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).
- **`CameraCutMoment`** — the hard cut that snaps to such a shot (no smooth transition — an instant, dramatic
  change).
- **`CameraFinished`** — the settled camera state after a sequence ends.

So moment cameras are the *punctuation marks* of the camera language — the special shots for the beats that matter
(finishing a race, being busted, [Chapter 48](../C48-Pursuit-Heat/04-bust-evade.md)). They're triggered by game
events ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) and bridge between the reactive gameplay
camera and the directed cinematic one ([C53.1](01-two-systems.md)) — a `CameraPhotoFinish` is neither pure gameplay
nor a full cutscene, but a scripted flourish at a transition.

> 🟡 *Reasoned:* the roles of `CameraPhotoFinish`/`CameraCutMoment` as event-triggered moment shots are inferred
> from their names and the race/transition context; the exact trigger wiring is deeper RE. The behaviour classes are
> verified.

## Why shared behaviours

Factoring camera effects into *shared behaviours* (rather than duplicating them in each camera system) is clean
composition ([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)):

- **Write once, use in both.** `CameraShake` is implemented once and used by the gameplay camera
  ([C53.2](02-cameraai.md)) *and* the director ([C53.3](03-cinematic-director.md)) — a crash shakes the view in
  both play and cutscene.
- **Composable punctuation.** A shot is a base framing (from `CameraAI` or a `CDAction*`) *plus* behaviours
  (shake, cut) layered on — the same composition pattern as mechanics and effects.
- **Consistent feel.** Because the behaviours are shared, a crash *feels* the same (same shake) whether you're
  driving or watching — the camera language is consistent across contexts.

So the shared behaviours are the camera system's *vocabulary of punctuation* — reusable effects and moment shots
that compose into both the reactive and directed cameras. They're what give MW's camera its *expressiveness*: the
shake of a crash, the cut to a finish, the orbit of a reveal — a shared toolkit the gameplay camera and the director
both draw on to make moments land.

## RE implications

- **Shared camera behaviours** — `CameraShake`, `CameraPhotoFinish`, `CameraCutMoment`, `CameraAnchor`,
  `CameraFromRacer` — compose into both camera systems.
- **`CameraShake`** punctuates impact (crash/nitrous/breaker) — the camera's channel of the multi-system feedback.
- **Moment cameras** (`CameraPhotoFinish`/`CameraCutMoment`) are event-triggered flourishes bridging gameplay and
  result.
- **Shared behaviours** — written once, used in both systems — consistent camera feel across contexts.

---

### Key takeaways

- **Reusable camera behaviours** — `CameraShake`, `CameraPhotoFinish`, `CameraCutMoment`, `CameraAnchor`,
  `CameraFromRacer` — compose into **both** the gameplay camera and the cinematic director.
- **`CameraShake`** punctuates violent moments (collision, nitrous, pursuit-breaker) — the camera's channel of the
  **multi-system feedback** (effects + sound + shake all reading one event).
- **Moment cameras** (`CameraPhotoFinish`, `CameraCutMoment`) are **event-triggered flourishes** for dramatic beats
  (race finish, bust) — bridging gameplay and cinematic.
- Factoring effects into **shared behaviours** means write-once, consistent feel across play and cutscene — the same
  composition pattern as mechanics and effects.
- They're the camera's **vocabulary of punctuation** — what make MW's moments land.

**Continue:** [C53.5 — Reading cameras in RE](05-reading-cameras.md) · [Chapter 53 hub](C53-Cameras-Director.md)
