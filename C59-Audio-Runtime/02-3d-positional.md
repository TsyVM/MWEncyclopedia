# C59.2 — 3D Positional Audio

> **The one-sentence version:** the `SFXCTL_3D*` controllers place each sound at a point in the world —
> `SFXCTL_3DCarPos`, `SFXCTL_3DLeftWheelPos`/`3DRightWheelPos`, `SFXCTL_3DHeliPos`, `SFXCTL_3DColPos` — so the
> mixer pans and attenuates it by where it is relative to the camera.

[← C59.1 — The runtime audio engine](01-audio-runtime.md) · [Chapter 59 hub](C59-Audio-Runtime.md) ·
[Next: C59.3 — Car sounds →](03-car-sounds.md)

---

## Sounds have positions

The defining feature of the audio runtime is that sounds are **3D-positioned** — each has a location in the world,
and the mixer spatialises it (pans left/right, attenuates by distance, applies Doppler). The verified `SFXCTL_3D*`
controllers are the *position sources* — 30 of them, one per kind of emitter:

| Controller | Emits at |
|---|---|
| `SFXCTL_3DCarPos` | the car's position |
| `SFXCTL_3DLeftWheelPos` / `3DRightWheelPos` | each wheel (tyre sound, [Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) |
| `SFXCTL_3DRearPos` | the rear (exhaust) |
| `SFXCTL_3DHeliPos` | the helicopter overhead |
| `SFXCTL_3DTrafficPos` | traffic cars |
| `SFXCTL_3DColPos` / `3DScrapePos` | collision / scrape points ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) |
| `SFXCTL_3DLeftWindPos` / `3DRightWindPos` | wind (per side) |
| `SFXCTL_3DTrailerPos` / `3DObjPos` / `3DFountainPos` | trailer / object / world emitters |

So *every* sound source has a controller giving its 3D position — the car, each wheel, the exhaust, the chopper,
each collision. The audio is thoroughly spatialised: not "a skid sound" but "a skid at the left wheel's position."

> ✅ *Verified:* the 30 `SFXCTL_3D*` positional controllers are present in `speed.exe` — including `3DCarPos`,
> `3DLeftWheelPos`/`3DRightWheelPos`, `3DHeliPos`, `3DTrafficPos`, `3DColPos`/`3DScrapePos`,
> `3DLeftWindPos`/`3DRightWindPos`, `3DTrailerPos`.

## Per-wheel positioning: the detail

A telling detail is the **per-wheel** controllers — `SFXCTL_3DLeftWheelPos` *and* `SFXCTL_3DRightWheelPos`
(separately). This means tyre sounds ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) are positioned *per
wheel*, not just "at the car":

- **A drift** ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) with the rear-left wheel spinning
  emits its screech at the *left wheel's* position — so it pans correctly as the car slides.
- **Two wheels on different surfaces** ([C44.1](../C44-Surfaces-Grip/01-surface-taxonomy.md)) — one on asphalt, one
  on grass — emit *different* road-noise sounds at *different* positions.
- **Wind** is even per-side (`3DLeftWindPos`/`3DRightWindPos`).

So the spatialisation is *fine-grained* — down to individual wheels and sides. This matches the per-wheel
simulation ([C39.3](../C39-Vehicle-Simulation/03-part-array.md), [C44.2](../C44-Surfaces-Grip/02-grip.md)): the
sim computes per-wheel state, and the audio positions per-wheel sound. The result is a soundscape that precisely
matches the physics — you *hear* which wheel is sliding, where the car is, where the cop is.

## The mixer spatialises

Given each sound's position ([above](#sounds-have-positions)), the **mixer** turns it into what you hear
([C59.1](01-audio-runtime.md)) relative to the camera ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)):

- **Pan** — a sound to your left plays louder on the left channel; behind you, in the rear (surround). A cop siren
  behind you *sounds* behind you.
- **Distance attenuation** — farther sounds are quieter; a distant chopper is faint, growing as it nears.
- **Doppler** — a fast-passing sound shifts pitch (the classic siren-pass effect).
- **Reverb** (`SFXObj_Reverb`, [C59.1](01-audio-runtime.md)) — environmental reverb by the space (a tunnel echoes).

So the mixer is where the *position* (from the controller) becomes *spatial audio*. The listener is the camera
([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)) — sounds are positioned relative to *where you're
watching from* — which is why the audio and the view are always consistent (a car you see on the right sounds on
the right). This camera-as-listener coupling ties the audio runtime to the camera system
([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)).

> 🟡 *Reasoned:* the pan/attenuation/Doppler/reverb spatialisation is the standard 3D-audio mixing model,
> consistent with the verified `SFXCTL_3D*` position controllers and `SFXObj_Reverb` bus; the exact mixer
> implementation is deeper RE. The positional controllers are verified.

## Why 3D-position everything

Positioning *every* sound in 3D (rather than flat stereo) is essential to the game's immersion:

- **Spatial awareness** — you *hear* where things are: a cop closing behind you (siren panning), a chopper
  overhead (`3DHeliPos`), traffic to the side. This is *gameplay-relevant* — the pursuit
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) is more legible when you can hear the threats' positions.
- **Consistency with the visuals** — the soundscape matches the view ([above](#the-mixer-spatialises)), so the
  world feels coherent and real.
- **Fine detail** — per-wheel ([above](#per-wheel-positioning-the-detail)) and per-side positioning makes drifts,
  surface changes, and impacts *audibly precise*.

So 3D positional audio is the sound counterpart of the 3D renderer — both place things in the world relative to
the camera. The `SFXCTL_3D*` controllers are what make MW's soundscape *spatial* rather than flat, and their
fine-grained coverage (per wheel, per collision point) is what makes it *precise*. Hearing the world is part of
seeing it.

## RE implications

- **`SFXCTL_3D*` controllers** (30) give each sound a 3D position — car, each wheel, heli, collision points, wind.
- **Per-wheel/per-side** positioning matches the per-wheel sim — you hear which wheel slides, on which surface.
- **The mixer spatialises** — pan, distance attenuation, Doppler, reverb — relative to the camera (the listener).
- **3D-positioning everything** buys spatial awareness (gameplay-relevant), visual consistency, and fine detail.

---

### Key takeaways

- Sounds are **3D-positioned** — the **`SFXCTL_3D*` controllers** (30) give each emitter a world position (car,
  each wheel, exhaust, heli, traffic, collision/scrape points, wind).
- Positioning is **fine-grained** — separate **`3DLeftWheelPos`/`3DRightWheelPos`** (and per-side wind) — matching
  the per-wheel sim, so you *hear* which wheel slides and on which surface.
- The **mixer spatialises** — pan, distance attenuation, Doppler, and reverb (`SFXObj_Reverb`) — relative to the
  **camera as listener** ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)).
- 3D positional audio gives **spatial awareness** (hear the cop behind, the chopper above — gameplay-relevant),
  **visual consistency**, and **fine detail**.
- It's the **sound counterpart of the 3D renderer** — both place the world relative to the camera; hearing is part
  of seeing.

**Continue:** [C59.3 — Car sounds](03-car-sounds.md) · [Chapter 59 hub](C59-Audio-Runtime.md)
