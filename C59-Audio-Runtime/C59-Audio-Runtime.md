# Chapter 59 — The Audio Runtime: SFX Controllers & 3D Mixing

> **Goal of this chapter:** decode the *runtime* audio engine (as opposed to the bank/codec data formats of
> Chapters 19–22) — the `SFXObj_*` mix buses, the `SFXCTL_3D*` positional controllers, the `CARSFX_*` per-car
> sound events, and the `Sound*` connectors that make every car, cop, and collision sound like itself in 3D
> space.

Chapters 19–22 decoded how audio is *stored* (banks, codecs, music, the GIN engine synth); this chapter decodes
how it's *played* — the live audio engine that positions sounds in 3D, mixes them through buses, and drives them
from the simulation. It's the runtime counterpart to the audio data: the system that turns "a car is drifting
here at this RPM" into the screech and engine note you hear, placed correctly in the stereo/surround field.

> **Verified against the executable.** The runtime audio is named in `speed.exe`: **`SFXObj_*` mix buses** (14) —
> `SFXObj_Ambience`, `SFXObj_Collision`, `SFXObj_Reverb`, `SFXObj_Speech`, `SFXObj_NISStream`, `SFXObj_FEHUD`,
> `SFXObj_PFEATrax`, `SFXObj_WorldObject`, …; **`SFXCTL_3D*` positional controllers** (30) — `SFXCTL_3DCarPos`,
> `SFXCTL_3DLeftWheelPos`/`3DRightWheelPos`, `SFXCTL_3DHeliPos`, `SFXCTL_3DTrafficPos`, `SFXCTL_3DColPos`
> (collision), `SFXCTL_3DScrapePos`, …; **`CARSFX_*` car sound events** (21) — `CARSFX_SingleGinsuEng`/
> `DualGinsuEng` (engine, [Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)), `CARSFX_Nitrous`,
> `CARSFX_Shift`, `CARSFX_Skids`, `CARSFX_Siren`, `CARSFX_RoadNoise`, `CARSFX_PreColWoosh`, …; **`Sound*`
> connectors** — `SoundConn`, `SoundCop`, `SoundRacer`, `SoundTraffic`, `SoundHeli`, `SoundAI`; and `SFXMasterVol`/
> `SFXModule`.

---

## Deep-dive pages

- [C59.1 — The runtime audio engine](01-audio-runtime.md): buses, controllers, events, connectors.
- [C59.2 — 3D positional audio](02-3d-positional.md): the `SFXCTL_3D*` controllers and the 3D mix.
- [C59.3 — Car sounds](03-car-sounds.md): the `CARSFX_*` events (engine, nitrous, skids, siren).
- [C59.4 — Sound connectors](04-sound-connectors.md): `SoundConn`/`SoundCop`/`SoundRacer` — per-entity audio.
- [C59.5 — Reading the audio runtime in RE](05-reading-audio.md): navigating the sound engine.

---

## 59.1 The runtime audio engine

The audio runtime ([C59.1](01-audio-runtime.md)) has four layers: **`SFXObj_*` buses** (mix groups — ambience,
collision, speech, music, HUD, reverb), **`SFXCTL_3D*` controllers** (where each sound emits in 3D,
[C59.2](02-3d-positional.md)), **`CARSFX_*` events** (the car sounds — engine, nitrous, skids,
[C59.3](03-car-sounds.md)), and **`Sound*` connectors** (per-entity audio, [C59.4](04-sound-connectors.md)). A
sound flows: an event (`CARSFX_*`) plays through a controller (`SFXCTL_3D*` for position) into a bus
(`SFXObj_*`), mixed to the output. `SFXMasterVol` is the master volume.

## 59.2 3D positional audio

The **`SFXCTL_3D*` controllers** ([C59.2](02-3d-positional.md)) place each sound *in the world* — `SFXCTL_3DCarPos`
(the car), `SFXCTL_3DLeftWheelPos`/`3DRightWheelPos` (each wheel's tyre sound,
[Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)), `SFXCTL_3DHeliPos` (the chopper overhead),
`SFXCTL_3DColPos`/`3DScrapePos` (collision/scrape points,
[Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)). So the audio is *spatialised*: a sound has a 3D
position, and the mixer pans/attenuates it by where it is relative to the camera
([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)) — a cop siren behind you sounds behind you.

## 59.3 Car sounds

The **`CARSFX_*` events** ([C59.3](03-car-sounds.md)) are the per-car sound vocabulary: `CARSFX_SingleGinsuEng`/
`DualGinsuEng` (the engine — "Ginsu" being the GIN granular synth,
[Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)), `CARSFX_Nitrous`, `CARSFX_Shift` (gear change),
`CARSFX_Skids` (tyre screech), `CARSFX_Siren` (cop), `CARSFX_RoadNoise` (surface,
[Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)), `CARSFX_PreColWoosh` (the pre-crash whoosh),
`CARSFX_TrafficHorn`. Each is driven by the sim state — the engine sound by RPM
([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)), the skids by tyre slip
([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)).

## 59.4 Sound connectors

Each entity's audio is a **`Sound*` connector** ([C59.4](04-sound-connectors.md)) — the audio counterpart of the
render/effect connectors ([C52.4](../C52-Effects-Particles/04-entity-effects.md)): `SoundCop` (a cop's siren +
engine), `SoundRacer` (a racer's), `SoundTraffic` (ambient cars), `SoundHeli` (the chopper), fed by `SoundConn`.
`SoundAI` ([C47.3](../C47-AI-Driver-Vehicle/03-managers.md)) is the fleet-level bridge (pursuit state → siren
intensity, music). So a car's sound *reads* its sim state (one-way, [C39.5](../C39-Vehicle-Simulation/05-connectors.md))
and plays the matching `CARSFX_*` events through 3D controllers.

---

### Key takeaways

- The runtime audio has four layers: **`SFXObj_*` buses** (mix groups), **`SFXCTL_3D*` controllers** (3D position),
  **`CARSFX_*` events** (car sounds), and **`Sound*` connectors** (per-entity).
- **3D positional audio** — `SFXCTL_3D*` places each sound in the world (car, each wheel, heli, collision points) —
  spatialised relative to the camera.
- **`CARSFX_*` events** are the car sound vocabulary — engine (`*GinsuEng`, the GIN synth), nitrous, shift, skids,
  siren, road noise — driven by sim state.
- **`Sound*` connectors** (`SoundCop`/`SoundRacer`/`SoundTraffic`/`SoundHeli`) are per-entity audio, reading the
  sim one-way, like the render/effect connectors.
- This is the **runtime** audio engine — distinct from the bank/codec/synth **data** formats
  ([Chapters 19](../C19-Audio-Banks/C19-Audio-Banks.md)–[22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).

**Next:** [Chapter 60 — Input Devices & Control Mapping](../C60-Input-Devices/C60-Input-Devices.md): how the player
commands the car.
