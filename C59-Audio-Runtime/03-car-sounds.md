# C59.3 — Car Sounds

> **The one-sentence version:** the `CARSFX_*` events are the per-car sound vocabulary — engine (`SingleGinsuEng`/
> `DualGinsuEng`, the GIN synth), nitrous, shift, skids, siren, road noise, the pre-crash whoosh — each driven by
> the car's sim state.

[← C59.2 — 3D positional audio](02-3d-positional.md) · [Chapter 59 hub](C59-Audio-Runtime.md) ·
[Next: C59.4 — Sound connectors →](04-sound-connectors.md)

---

## The car sound vocabulary

A car makes many sounds, and the verified `CARSFX_*` events are its *vocabulary* — 21 named car-sound events:

| Event | Sound | Driven by |
|---|---|---|
| `CARSFX_SingleGinsuEng` / `DualGinsuEng` | the engine | RPM ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) |
| `CARSFX_AEMSEngine` / `TrafficEngine` | alt engine models | RPM |
| `CARSFX_Nitrous` | NOS surge | nitrous state ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `CARSFX_Shift` | gear change | the shift ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `CARSFX_Skids` | tyre screech | tyre slip ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) |
| `CARSFX_RoadNoise` | tyre roll | surface ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) |
| `CARSFX_Siren` | cop siren | cop state ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) |
| `CARSFX_TrafficHorn` | traffic horn | traffic AI |
| `CARSFX_BottomOut` | suspension bottoming | suspension ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)) |
| `CARSFX_SparkChatter` | scrape sparks | collision ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) |
| `CARSFX_PreColWoosh` | pre-crash whoosh | imminent collision |
| `CARSFX_Rain` | rain on the car | weather ([C57.3](../C57-World-Systems/03-weather-rain.md)) |

So every audible aspect of a car — engine, transmission, tyres, suspension, collisions, siren, weather — has a
`CARSFX_*` event. The car's soundscape is these events, played and mixed as the sim drives them.

> ✅ *Verified:* the `CARSFX_*` events (21) are present in `speed.exe` — `SingleGinsuEng`/`DualGinsuEng`,
> `AEMSEngine`, `Nitrous`, `Shift`, `Skids`, `RoadNoise`, `Siren`, `TrafficHorn`, `BottomOut`, `SparkChatter`,
> `PreColWoosh`, `Rain`, `TrafficEngine`.

## Ginsu: the engine sound

The engine events **`CARSFX_SingleGinsuEng`/`DualGinsuEng`** confirm the engine-sound system
([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)): "**Ginsu**" is the GIN granular synth
([C22.1](../C22-Engine-Sound-GIN/01-granular-synthesis.md)), and *Single*/*Dual* likely denote one or two synth
voices (a single engine note vs. a layered dual-voice for richer engines):

- **The synth is driven by RPM** ([C22.4](../C22-Engine-Sound-GIN/04-rpm-bridge.md)) — the engine mechanic's RPM
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) sweeps the GIN grains
  ([C22.3](../C22-Engine-Sound-GIN/03-grains.md)) across the rpm range (`Gnsu` header,
  [C22.2](../C22-Engine-Sound-GIN/02-gnsu-header.md)).
- **`CARSFX_SingleGinsuEng`** is the runtime *event* that plays this synth — the bridge from the car's engine state
  to the GIN synth output ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).
- **Dual** adds a second voice for a fuller sound (a V8's layered note vs. a four-cylinder's single).

So `CARSFX_*GinsuEng` is where the *runtime* (this chapter) meets the *synth data*
([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)): the event plays the GIN synth, driven by RPM,
positioned at the car ([C59.2](02-3d-positional.md)), on the engine bus. The famous MW engine note is
`CARSFX_SingleGinsuEng`/`DualGinsuEng` running the GIN granular synth on the car's live RPM.

## Sim-driven: sound tracks physics

The crucial property of the `CARSFX_*` events is that they're **sim-driven** — each reads a piece of the car's
physics ([above](#the-car-sound-vocabulary)) and plays accordingly ([C59.4](04-sound-connectors.md)):

- **Engine** tracks RPM ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) — the note rises with
  revs, drops on a shift (`CARSFX_Shift`).
- **Skids** track tyre slip ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) — the screech starts
  when the tyre breaks loose, on the surface ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)).
- **Nitrous** plays when NOS fires ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)); `BottomOut`
  when the suspension bottoms ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)).
- **PreColWoosh** anticipates a crash — a whoosh *before* impact, cueing the collision.

So the car's sound is a *live readout* of its physics — you hear the engine's revs, the tyres' grip, the
suspension's compression, the imminent crash. This is why MW's cars sound *alive*: the audio is driven by the same
sim state ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) as the visuals
([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)), so sound and motion are locked together. The
`CARSFX_*` events are the sound mechanic ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) reading the sim
and speaking the car's state.

## RE implications

- **`CARSFX_*` events** (21) are the car sound vocabulary — engine, nitrous, shift, skids, road noise, siren,
  bottom-out, sparks, pre-crash whoosh.
- **`*GinsuEng`** plays the GIN granular synth ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) —
  Single/Dual voices, driven by RPM — where the runtime meets the synth data.
- **Sim-driven** — each event reads a piece of physics (RPM, slip, NOS, suspension, imminent collision).
- **Sound tracks physics** — the car sounds alive because the audio and visuals share the sim state.

---

### Key takeaways

- The **`CARSFX_*` events** (21) are a car's **sound vocabulary** — engine, nitrous, shift, skids, road noise,
  siren, bottom-out, sparks, pre-crash whoosh, rain.
- **`CARSFX_SingleGinsuEng`/`DualGinsuEng`** play the **GIN granular synth**
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) — *Single*/*Dual* voices — driven by RPM: MW's
  famous engine note, where the runtime meets the synth data.
- Every event is **sim-driven** — engine tracks RPM, skids track slip, nitrous fires with NOS, bottom-out with
  suspension, the whoosh anticipates a crash.
- The car sounds **alive** because its audio reads the **same sim state** as the visuals — sound and motion locked
  together.
- The `CARSFX_*` events are the **SOUND mechanic** ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) — the
  car speaking its physical state.

**Continue:** [C59.4 — Sound connectors](04-sound-connectors.md) · [Chapter 59 hub](C59-Audio-Runtime.md)
