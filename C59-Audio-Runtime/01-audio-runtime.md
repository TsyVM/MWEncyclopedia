# C59.1 — The Runtime Audio Engine

> **The one-sentence version:** the audio runtime is four layers — `SFXObj_*` mix buses, `SFXCTL_3D*` positional
> controllers, `CARSFX_*` sound events, and `Sound*` connectors — through which a sim event becomes a positioned,
> mixed sound.

[← Chapter 59 hub](C59-Audio-Runtime.md) · [Next: C59.2 — 3D positional audio →](02-3d-positional.md)

---

## Four layers

The runtime audio engine turns simulation into sound through four verified layers:

```
sim event (e.g. car drifting, RPM 6000, on asphalt)
   → CARSFX_* event         — WHAT sound (skid, engine note)      [C59.3]
      → SFXCTL_3D* controller — WHERE it emits (the wheel's position) [C59.2]
         → SFXObj_* bus       — WHICH mix group (ambience/collision) [below]
            → the mixer       — pan/attenuate by camera, apply reverb
               → the output   — stereo/surround
```

So a sound has an *identity* (the event), a *position* (the controller), and a *bus* (the mix group). This
separation — what, where, which-group — is the structure of the whole audio runtime, and each layer is a set of
verified classes ([C59 hub](C59-Audio-Runtime.md)).

> ✅ *Verified:* the four layers are named in `speed.exe` — `SFXObj_*` (14 buses), `SFXCTL_3D*` (30 controllers),
> `CARSFX_*` (21 events), `Sound*` connectors; plus `SFXMasterVol` and `SFXModule`.

## The SFXObj buses

The **`SFXObj_*` classes** are the *mix buses* — the groups sounds are routed into for mixing
([C59.5](05-reading-audio.md)):

| Bus | Sounds |
|---|---|
| `SFXObj_Ambience` | world ambience (wind, city hum) |
| `SFXObj_Collision` | crashes, scrapes ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) |
| `SFXObj_Speech` | cop/character voice ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) |
| `SFXObj_PFEATrax` | music ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) |
| `SFXObj_Reverb` | the reverb send/bus |
| `SFXObj_FEHUD` | front-end/HUD sounds |
| `SFXObj_NISStream`/`MomentStrm` | cutscene streams ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)) |
| `SFXObj_Helicopter`/`WorldObject`/`Woosh`/`TruckFX` | specific effect groups |

So the audio is organised into *buses by category* — ambience, collision, speech, music, HUD — each mixable
independently (volume, reverb). This is the standard audio-mixing architecture: sounds route to buses, buses to
the master. `SFXMasterVol` is the master volume; a bus like `SFXObj_Reverb` applies a shared effect. Grouping by
bus lets the mix be balanced per category (duck the music under speech, [C59.5](05-reading-audio.md)) without
touching individual sounds.

## The controllers and events

The other two layers ([C59.2](02-3d-positional.md)–[C59.3](03-car-sounds.md)):

- **`SFXCTL_3D*` controllers** ([C59.2](02-3d-positional.md)) — the *positional* layer: each names a *place* a
  sound emits (`SFXCTL_3DCarPos`, `SFXCTL_3DLeftWheelPos`). The controller feeds the mixer the 3D position for
  panning/attenuation.
- **`CARSFX_*` events** ([C59.3](03-car-sounds.md)) — the *content* layer: each names a *sound* (`CARSFX_Skids`,
  `CARSFX_Nitrous`). The event selects the sample/synth ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).

So a playing sound is an *event* at a *controller's position* on a *bus* — e.g. `CARSFX_Skids` at
`SFXCTL_3DLeftWheelPos` on `SFXObj_Collision`(-adjacent). The three combine into one positioned, categorised
sound. This compositional structure ([C59 hub](C59-Audio-Runtime.md)) is why the audio can represent so many
distinct sounds from a small set of parts — events × positions × buses.

## Why a layered audio runtime

The layered design ([above](#four-layers)) serves the same goals as the rest of the engine
([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)):

- **Composition** — a sound is composed from an event + position + bus, so new sounds reuse the layers (a new
  car-sound is a new `CARSFX_*` event through existing controllers/buses).
- **Independent mixing** — the buses ([above](#the-sfxobj-buses)) let the mix be balanced per category without
  per-sound work.
- **Spatial correctness** — the controllers ([C59.2](02-3d-positional.md)) place every sound in 3D, so the
  soundscape matches the visuals.
- **Sim-driven** — the connectors ([C59.4](04-sound-connectors.md)) feed sim state to the events, so sounds track
  gameplay (RPM, slip, collisions).

So the audio runtime is the *presentation* layer for sound ([C52.5](../C52-Effects-Particles/05-reading-effects.md))
— composed, mixed, spatialised, and sim-driven — the aural counterpart to the renderer
([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) and effects
([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)). It reads the sim and produces the soundscape,
never perturbing the physics.

## RE implications

- **The audio runtime is four layers** — `SFXObj_*` buses, `SFXCTL_3D*` controllers, `CARSFX_*` events, `Sound*`
  connectors.
- **`SFXObj_*` buses** group sounds by category (ambience/collision/speech/music) for independent mixing.
- **A sound is an event × position × bus** — composed from the layers.
- **Layered design** buys composition, independent mixing, spatial correctness, and sim-driven sound.

---

### Key takeaways

- The runtime audio engine is **four layers**: **`SFXObj_*` mix buses**, **`SFXCTL_3D*` positional controllers**,
  **`CARSFX_*` sound events**, and **`Sound*` connectors** — verified in `speed.exe`.
- A playing sound is an **event × controller-position × bus** — composed from the layers (what, where, which
  group).
- **`SFXObj_*` buses** group sounds by category (ambience, collision, speech, music, HUD, reverb) for independent
  mixing under `SFXMasterVol`.
- The layered design buys **composition** (reuse the layers), **independent mixing** (per bus), **spatial
  correctness** (per controller), and **sim-driven** sound (per connector).
- The audio runtime is the **presentation layer for sound** — reading the sim and producing the soundscape, like
  the renderer and effects.

**Continue:** [C59.2 — 3D positional audio](02-3d-positional.md) · [Chapter 59 hub](C59-Audio-Runtime.md)
