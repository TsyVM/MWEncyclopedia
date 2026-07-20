# C59.5 — Reading the Audio Runtime in RE

> **The one-sentence version:** navigate the audio runtime by the `SFXObj_*` buses, the `SFXCTL_3D*` controllers,
> the `CARSFX_*` events, and the `Sound*` connectors — reading the sound engine as buses × positions × events,
> driven from the sim.

[← C59.4 — Sound connectors](04-sound-connectors.md) · [Chapter 59 hub](C59-Audio-Runtime.md) ·
[Next: Chapter 60 — Input Devices & Control Mapping →](../C60-Input-Devices/C60-Input-Devices.md)

---

## Anchors for audio-runtime RE

The audio runtime is anchored on verified string families:

- **The buses** — `SFXObj_*` (14) ([C59.1](01-audio-runtime.md)).
- **The controllers** — `SFXCTL_3D*` (30) ([C59.2](02-3d-positional.md)).
- **The events** — `CARSFX_*` (21) ([C59.3](03-car-sounds.md)).
- **The connectors** — `Sound*` (`SoundCop`/`SoundRacer`/…), `SoundAI` ([C59.4](04-sound-connectors.md)).
- **The master** — `SFXMasterVol`, `SFXModule`.

From these, the audio runtime is navigable: the buses, the positions, the events, and the per-entity behaviour.

## The RE workflow

Reading the audio runtime:

1. **Map the buses** — the `SFXObj_*` mix groups ([C59.1](01-audio-runtime.md)).
2. **Map the controllers** — the `SFXCTL_3D*` positions ([C59.2](02-3d-positional.md)).
3. **Map the events** — the `CARSFX_*` sounds and their sim drivers ([C59.3](03-car-sounds.md)).
4. **Trace the connectors** — how `Sound*` reads the sim and plays events ([C59.4](04-sound-connectors.md)).

The output is the full audio picture: buses, positions, events, and per-entity behaviour.

## The naming is the documentation

Like the cop system ([C49.6](../C49-Cops-Dispatch-Roadblocks/06-reading-fleet.md)), the audio runtime is
*self-documenting* — the string names spell out the whole system:

- **`SFXObj_Collision`, `SFXObj_Speech`, `SFXObj_Reverb`** — the buses name their categories.
- **`SFXCTL_3DLeftWheelPos`, `SFXCTL_3DHeliPos`** — the controllers name their emitter positions.
- **`CARSFX_Nitrous`, `CARSFX_Skids`, `CARSFX_SingleGinsuEng`** — the events name their sounds.

So reverse-engineering the audio is largely *reading the names* — grep `SFXObj_`, `SFXCTL_3D`, `CARSFX_` and you
have the buses, positions, and events. The system labelled itself ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)):
the naming *is* the specification. This is the string-verification technique
([C50.3](../C50-Verification-Methodology/03-hash-verification.md)) at its easiest — no hashing needed, the strings
are right there, and their structured prefixes (`SFXObj_`/`SFXCTL_`/`CARSFX_`) map the architecture directly.

## The audio ties to the sim and camera

The audio runtime connects to two systems the book already decoded:

- **The sim** ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) — the `Sound*` connectors
  ([C59.4](04-sound-connectors.md)) read the physics (RPM, slip, collisions) that drive the `CARSFX_*` events
  ([C59.3](03-car-sounds.md)). Sound is a sim reader.
- **The camera** ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)) — the listener is the camera
  ([C59.2](02-3d-positional.md)); the mixer spatialises relative to where you're viewing from. Sound is
  camera-relative.

So the audio sits between the sim (what makes the sound) and the camera (where you hear it from) — a reader of the
physics, spatialised to the view. This dual coupling ([C59.2](02-3d-positional.md), [C59.4](04-sound-connectors.md))
is what makes the soundscape both *accurate* (matching the physics) and *coherent* (matching the view). Reading the
audio runtime completes the presentation trio — render ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)),
effects ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)), and sound (this chapter) — all reading
one sim, all relative to one camera.

## RE implications

- **Anchor on** the `SFXObj_*` buses, `SFXCTL_3D*` controllers, `CARSFX_*` events, and `Sound*` connectors.
- **The RE workflow** — map buses → controllers → events → connectors.
- **The naming is the documentation** — grep the structured prefixes to recover the whole system.
- **Audio ties to sim and camera** — a physics reader, spatialised to the view — completing the presentation trio.

---

### Key takeaways

- The audio runtime is anchored on the **`SFXObj_*` buses**, **`SFXCTL_3D*` controllers**, **`CARSFX_*` events**,
  and **`Sound*` connectors** — all verified string families.
- The RE workflow: **map the buses → controllers → events → connectors**.
- The audio is **self-documenting** — the structured prefixes (`SFXObj_`/`SFXCTL_3D`/`CARSFX_`) grep directly to
  the buses, positions, and events; the naming *is* the specification.
- The audio ties to the **sim** (the `Sound*` connectors read physics that drive the events) and the **camera**
  (the listener; sounds spatialised to the view).
- Reading the audio completes the **presentation trio** — render, effects, and sound — all one-way readers of one
  sim, relative to one camera.

**Next:** [Chapter 60 — Input Devices & Control Mapping](../C60-Input-Devices/C60-Input-Devices.md): how the player
commands the car.

**Sources:** `speed.exe` (verified: `SFXObj_*` buses — `Ambience`/`Collision`/`Speech`/`Reverb`/`PFEATrax`/`FEHUD`/
`NISStream`/`WorldObject`/…; `SFXCTL_3D*` controllers — `3DCarPos`/`3DLeftWheelPos`/`3DRightWheelPos`/`3DHeliPos`/
`3DTrafficPos`/`3DColPos`/`3DScrapePos`/…; `CARSFX_*` events — `SingleGinsuEng`/`DualGinsuEng`/`Nitrous`/`Shift`/
`Skids`/`Siren`/`RoadNoise`/`BottomOut`/`SparkChatter`/`PreColWoosh`/…; `Sound*` connectors — `SoundConn`/`SoundCop`/
`SoundRacer`/`SoundTraffic`/`SoundHeli`/`SoundAI`; `SFXMasterVol`/`SFXModule`).
