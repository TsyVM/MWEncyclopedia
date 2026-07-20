# C57.2 — Sky & Fog

> **The one-sentence version:** the world is framed by a skybox (`SkyColor`, `SkyBlendFactor`, `skyshader`) drawn
> behind everything and atmospheric fog (`FogShader`, `FogValue`) that fades distant geometry — the backdrop and
> the depth-haze that give Rockport its sense of space and air.

[← C57.1 — Time of day & the sun](01-time-of-day.md) · [Chapter 57 hub](C57-World-Systems.md) ·
[Next: C57.3 — Weather & rain →](03-weather-rain.md)

---

## The skybox

Behind all the geometry is the **skybox** — the distant sky drawn as the world's backdrop. The verified components:

- **`skybox` / `SkyboxCurrentGen`** — the sky geometry/system (a large dome or box textured with the sky).
- **`SkyColor` / `skyrgba`** — the sky's colour, which shifts with time ([C57.1](01-time-of-day.md)) — warm at the
  golden hour, cooler otherwise.
- **`SkyBlendFactor`** — blends between sky states (e.g. transitioning sky colours/textures).
- **`skyshader` / `SkyTextureMisc`** — the shader and textures that render the sky.

So the sky is a *rendered backdrop* — not just a flat colour, but a shaded, textured, colour-shifting dome behind
the world. It sets the *upper* half of every frame and the distant horizon, contributing hugely to the world's mood
(a warm golden sky vs. a grey one changes everything). The `SkyColor` tracks the `TimeOfDay`
([C57.1](01-time-of-day.md)) — the sky is part of the golden-hour look.

> ✅ *Verified:* the sky system — `skybox`, `SkyboxCurrentGen`, `SkyColor`, `skyrgba`, `SkyBlendFactor`,
> `skyshader`, `SkyTextureMisc` — is present in `speed.exe`.

## Fog: the atmosphere of depth

**Fog** (`FogShader`, `FogValue`) is the atmospheric haze that fades geometry with distance — and it does two jobs:

- **Atmosphere** — the haze gives the world *air* and *depth*; distant buildings fade into the horizon, making the
  city feel vast and real (rather than sharply drawn to infinity).
- **Draw-distance hiding** — fog conveniently *hides the far draw distance* ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md),
  [Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) — geometry fades into fog *before* it would pop in/out at
  the streaming boundary, so the cull ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) is masked by the haze.

So fog is both *aesthetic* (depth atmosphere) and *practical* (hiding the draw-distance limit). `FogValue` sets how
thick it is — more fog = closer horizon, more hidden pop-in, but also a heavier atmosphere. This dual role
(atmosphere + draw-distance mask) is a classic rendering technique ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md))
— the fog that makes the world feel deep is *also* what lets the renderer stop drawing distant geometry gracefully.
The warm golden fog of MW's atmosphere is a perfect example: it looks like sun-haze *and* masks the streaming
horizon.

> 🟡 *Reasoned:* the fog's draw-distance-hiding role is the standard use of distance fog with a streaming world,
> consistent with the verified `FogShader`/`FogValue` and the cull/streaming systems
> ([Chapters 15](../C15-Track-Streaming/C15-Track-Streaming.md)–[16](../C16-Scenery-Cull/C16-Scenery-Cull.md)); the
> exact fog parameters are per-world data. The fog system is verified.

## Sky and fog together: the frame's air

The sky and fog together give the world its *atmosphere* — the sense of being in a real, airy space:

- **The sky is the *far* backdrop** — the dome behind everything, the horizon colour.
- **The fog is the *near-to-far* transition** — the haze that connects the drawn geometry to the distant sky.

So looking down a Rockport street, you see: the sharp near geometry, fading through the fog haze
([above](#fog-the-atmosphere-of-depth)), into the distant skybox ([above](#the-skybox)) at the horizon. The fog is
the *blend* between the world and the sky — the atmospheric gradient that makes the transition seamless (no hard
edge where geometry ends and sky begins). This is what gives the world *air* and *depth* — the combination of a
coloured sky and distance fog, both tinted by the golden hour ([C57.1](01-time-of-day.md)). Together they frame
every shot, setting the mood before the cars and treatment ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md))
add theirs.

## RE implications

- **The skybox** (`skybox`/`SkyColor`/`skyshader`) is the rendered distant backdrop — colour tracking `TimeOfDay`.
- **Fog** (`FogShader`/`FogValue`) fades distant geometry — atmosphere *and* draw-distance hiding.
- **Together** they give the world *air and depth* — near geometry fades through fog into the sky.
- **Both tint golden** ([C57.1](01-time-of-day.md)) — part of MW's warm atmospheric identity.

---

### Key takeaways

- The world is framed by a **skybox** — `skybox`/`SkyColor`/`SkyBlendFactor`/`skyshader` — a shaded, textured,
  colour-shifting backdrop dome whose colour tracks the `TimeOfDay` ([C57.1](01-time-of-day.md)).
- **Fog** (`FogShader`/`FogValue`) fades distant geometry — doing double duty as **atmosphere** (depth/air) and a
  **draw-distance mask** (hiding the streaming/cull horizon,
  [Chapters 15](../C15-Track-Streaming/C15-Track-Streaming.md)–[16](../C16-Scenery-Cull/C16-Scenery-Cull.md)).
- Together, sky and fog give the world **air and depth** — near geometry fades through the fog haze into the distant
  sky, seamlessly.
- Both are **tinted by the golden hour** — part of MW's warm atmospheric identity, set before the cars and
  treatment.
- The fog is the classic **atmosphere + practical** technique — it looks like sun-haze *and* gracefully masks the
  draw distance.

**Continue:** [C57.3 — Weather & rain](03-weather-rain.md) · [Chapter 57 hub](C57-World-Systems.md)
