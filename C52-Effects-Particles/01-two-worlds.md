# C52.1 — The Two Effect Worlds

> **The one-sentence version:** Most Wanted has two distinct effect systems — 3D **scene particles** (smoke,
> sparks, dust spawned into the world by emitters) and 2D **screen post-process** (brightness, radial blur,
> vignette applied to the whole frame) — and the post-process components build the `VisualTreatment`.

[← Chapter 52 hub](C52-Effects-Particles.md) · [Next: C52.2 — Emitters & particles →](02-emitters-particles.md)

---

## Two kinds of "effect"

The word "effect" covers two fundamentally different things in the renderer, and MW has verified classes for both:

- **Scene particles (3D).** Things spawned *into the world* at a position — tyre smoke, impact sparks, road dust,
  debris. They have 3D positions, move, and are drawn as part of the scene ([C52.2](02-emitters-particles.md)).
  Verified: `EmitterSlotPool`, `ParticleSlotPool`, `ParticlesShader`.
- **Screen post-process (2D).** Filters applied to the *whole rendered image* — brightness, radial (speed) blur,
  vignette. They have no 3D position; they operate on the final 2D frame
  ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)). Verified: `EffectBrightness`, `EffectRadialBlur`,
  `EffectVignette`.

These are different in *kind*: a particle is an object in the world; a post-process is a transformation of the
picture. Conflating them is a common confusion — the chapter separates them because they're built and used
differently.

> ✅ *Verified:* the two systems are distinct classes in `speed.exe` — scene particles (`Emitter*`/`Particle*`
> pools, `ParticlesShader`) and screen post-process (`EffectBrightness`/`EffectRadialBlur`/`EffectVignette`,
> `EffectTarget`).

## Scene particles: things in the world

Scene particles ([C52.2](02-emitters-particles.md)) are **3D world objects** with a lifecycle:

- **Spawned** by an emitter at a world position (a wheel contact, an impact point).
- **Simulated** — they move (rise, drift, fall), age, and fade over a lifetime.
- **Drawn** into the scene by `ParticlesShader`, so they're occluded by geometry, lit by the world, and part of the
  3D image.

So a puff of tyre smoke is a cloud of particles born at the wheel, drifting up and back as the car passes, fading
out — real objects in the world, drawn among the buildings and cars. This is the effect world that makes the game
feel *alive* and *physical* — the smoke, dust, and sparks that react to what you do.

## Screen post-process: filtering the image

Screen post-process effects ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) are **2D image filters** with
no world presence:

- **`EffectBrightness`** — adjusts the frame's brightness/exposure (part of the colour grade).
- **`EffectRadialBlur`** — blurs radially from the centre — the *speed blur* that intensifies with velocity/nitrous
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)).
- **`EffectVignette`** — darkens the frame edges — the gritty focus.
- **`EffectTarget`** — the render target the post-process reads/writes.

These compose the **`VisualTreatment`** ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) — MW's signature
look is *built from* these components. Radial blur + vignette + brightness grading, driven by state, *is* the
treatment. So the post-process world is the *mood* layer — it doesn't add objects, it transforms the whole image's
feel.

## Why separate systems

The two effect worlds are separate systems because they solve different problems with different techniques:

- **Particles need world simulation** — position, velocity, lifetime, collision, occlusion. They're a mini
  physics/render system for many small sprites ([C52.2](02-emitters-particles.md)).
- **Post-process needs full-frame passes** — reading the rendered image as a texture and transforming it
  ([C51.5](../C51-Render-Pipeline/05-render-frame.md)). They're screen-space shader passes.

They also run at *different points* in the frame ([C51.5](../C51-Render-Pipeline/05-render-frame.md)): particles are
drawn *during* the scene render (they're in the world); post-process runs *after* (on the finished image). So the
frame is: draw scene (including particles) → post-process (the treatment) → present. Understanding that particles
are *in* the scene and post-process is *over* it is the key to reading the effect systems correctly — they're two
layers, world and image.

## RE implications

- **Two effect worlds** — 3D scene particles (`Emitter*`/`Particle*`) and 2D screen post-process
  (`Effect*Blur`/`Vignette`).
- **Particles** are world objects — spawned, simulated, drawn *in* the scene (smoke, sparks, dust).
- **Post-process** filters the whole image — `EffectRadialBlur`/`Vignette`/`Brightness` compose the
  `VisualTreatment`.
- **Different systems, different frame points** — particles during the scene render, post-process after.

---

### Key takeaways

- MW has **two distinct effect systems**: 3D **scene particles** (smoke/sparks/dust *in* the world) and 2D **screen
  post-process** (filters *over* the whole image).
- **Scene particles** are world objects with a lifecycle — spawned by emitters, simulated (move/age/fade), drawn by
  `ParticlesShader` among the geometry.
- **Screen post-process** — `EffectBrightness`/`EffectRadialBlur`/`EffectVignette` — are the composable **building
  blocks of the `VisualTreatment`** ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).
- They run at **different frame points** — particles *during* the scene render, post-process *after* (on the
  finished image).
- Reading effects correctly means keeping the two apart: particles are **in** the scene, post-process is **over**
  it.

**Continue:** [C52.2 — Emitters & particles](02-emitters-particles.md) · [Chapter 52 hub](C52-Effects-Particles.md)
