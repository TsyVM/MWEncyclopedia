# C59.4 — Sound Connectors

> **The one-sentence version:** each entity's audio is a `Sound*` connector — `SoundCop` (siren + engine),
> `SoundRacer`, `SoundTraffic`, `SoundHeli` — fed by `SoundConn`, reading the entity's sim state one-way, with
> `SoundAI` as the fleet-level pursuit→audio bridge.

[← C59.3 — Car sounds](03-car-sounds.md) · [Chapter 59 hub](C59-Audio-Runtime.md) ·
[Next: C59.5 — Reading the audio runtime in RE →](05-reading-audio.md)

---

## Per-entity sound connectors

Each sound-making entity has a **`Sound*` connector** — the audio counterpart of the render
([C51.2](../C51-Render-Pipeline/02-render-objects.md)) and effect
([C52.4](../C52-Effects-Particles/04-entity-effects.md)) connectors. The verified set:

| Connector | Entity | Sounds |
|---|---|---|
| `SoundCop` | a cop car | siren, engine, pursuit chatter |
| `SoundRacer` | a racer | engine, tyres |
| `SoundTraffic` | ambient traffic | engine, horn |
| `SoundHeli` | the helicopter | rotor, spotlight |
| `SoundFX` | effects | impact/environmental |
| `SoundConn` | (base) | the connector itself |

So each entity type has its *own* sound connector, which knows *which* `CARSFX_*` events
([C59.3](03-car-sounds.md)) that entity plays and *when*. A `SoundCop` plays the siren (`CARSFX_Siren`) and the
cop engine; a `SoundTraffic` plays a traffic engine and horn. The connector is the per-entity audio *behaviour*.

> ✅ *Verified:* the `Sound*` connectors — `SoundConn`, `SoundCop`, `SoundRacer`, `SoundTraffic`, `SoundHeli`,
> `SoundFX`, `SoundID` — are present in `speed.exe`; `SoundAI` ([C47.3](../C47-AI-Driver-Vehicle/03-managers.md))
> is the fleet-level bridge.

## SoundConn: sim state to sound

The **`SoundConn`** connector ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) is the one-way bridge from an
entity's sim state to its sound — exactly parallel to `RenderConn` ([C51.2](../C51-Render-Pipeline/02-render-objects.md))
and `EffectConn` ([C52.4](../C52-Effects-Particles/04-entity-effects.md)):

```
entity (a cop car) publishes its state:
   RPM, speed, tyre slip, siren-on, damage
      → SoundConn reads it (one-way)
         → SoundCop decides which CARSFX_* events to play
            → events play through 3D controllers on buses (C59.1-3)
```

So the flow is *entity state → `SoundConn` → `Sound*` connector → `CARSFX_*` events → mix*
([C59.1](01-audio-runtime.md)). The connector *reads* the sim ([C39.5](../C39-Vehicle-Simulation/05-connectors.md))
and never perturbs it — the audio is a pure consumer of physics, like the renderer and effects. This is the *third*
of the presentation connectors ([C52.5](../C52-Effects-Particles/05-reading-effects.md)): render (draw), effect
(particles), sound (audio) — all one-way readers of the same sim state. A car's look, effects, and sound are three
connectors reading one physics.

## SoundCop vs. SoundRacer vs. SoundTraffic

That each entity type has its *own* connector (`SoundCop`, `SoundRacer`, `SoundTraffic`) reflects their different
sound needs ([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)):

- **`SoundCop`** — the cop's audio: the *siren* (`CARSFX_Siren`, its defining sound), the pursuit engine, and the
  cop chatter ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)). A cop *sounds* like a cop.
- **`SoundRacer`** — a racer's audio: engine and tyres, tuned for a performance car
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) — the aggressive note of a
  rival.
- **`SoundTraffic`** — ambient traffic: a cheaper engine, the occasional horn — background sound, not foreground.
- **`SoundHeli`** — the chopper's rotor and spotlight — the overhead threat's audio.

So the per-entity connectors give each kind of car its *characteristic* soundscape — the siren-wailing cop, the
snarling rival, the mundane traffic — from the shared `CARSFX_*` vocabulary
([C59.3](03-car-sounds.md)). This is the audio equivalent of the per-entity effects
([C52.4](../C52-Effects-Particles/04-entity-effects.md)): each entity type knows its own presentation, composed
from shared parts.

## SoundAI: the fleet bridge

Above the per-car connectors is **`SoundAI`** ([C47.3](../C47-AI-Driver-Vehicle/03-managers.md)) — the
*fleet-level* audio bridge that translates *pursuit state* into the overall soundscape:

- **Siren intensity** — as the pursuit escalates ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), the
  chorus of sirens grows (more cops, [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).
- **Music intensity** — the pursuit drives the music ([Chapter 66](../C66-Interactive-Music/C66-Interactive-Music.md))
  — tenser as Heat rises.
- **Callouts** — the cop chatter ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) ("suspect spotted," "roadblock
  ahead") is triggered by pursuit events.

So `SoundAI` is the audio counterpart of the pursuit director ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md))
— it reads the *fleet/chase* state (not just one car) and shapes the *whole* pursuit soundscape: the wall of
sirens, the intensifying music, the radio chatter. This is why a high-Heat pursuit *sounds* overwhelming — `SoundAI`
scales the audio with the chase. It's the bridge from the pursuit system to the ears, completing the sim→sound
chain at the fleet level.

## RE implications

- **Per-entity `Sound*` connectors** — `SoundCop`/`SoundRacer`/`SoundTraffic`/`SoundHeli` — each entity's audio
  behaviour.
- **`SoundConn`** feeds sim state to sound (one-way) — the third presentation connector (render/effect/sound).
- **Per-entity character** — cop (siren), racer (snarl), traffic (mundane) — from the shared `CARSFX_*` vocabulary.
- **`SoundAI`** is the fleet bridge — pursuit state → siren chorus, music intensity, chatter.

---

### Key takeaways

- Each entity's audio is a **`Sound*` connector** — `SoundCop` (siren + engine), `SoundRacer`, `SoundTraffic`,
  `SoundHeli` — knowing which `CARSFX_*` events that entity plays.
- **`SoundConn`** feeds entity sim state to sound **one-way** — the **third presentation connector** alongside
  `RenderConn` and `EffectConn` (a car's look, effects, and sound are three readers of one physics).
- **Per-entity connectors** give each car type its **characteristic soundscape** — the siren-wailing cop, the
  snarling rival, the mundane traffic — from a shared vocabulary.
- **`SoundAI`** is the **fleet-level bridge** — it scales the *whole* pursuit soundscape (siren chorus, music
  intensity, radio chatter) with the chase state ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- This is why a high-Heat pursuit **sounds overwhelming** — `SoundAI` grows the audio with the Heat.

**Continue:** [C59.5 — Reading the audio runtime in RE](05-reading-audio.md) · [Chapter 59 hub](C59-Audio-Runtime.md)
