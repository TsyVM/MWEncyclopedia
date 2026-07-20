# C57.4 — World Lighting

> **The one-sentence version:** time, sky, and weather all feed the world lighting — the `SunColor` sets the key
> light, the sky the ambient, the fog the atmosphere — applied to surfaces via `LightMaterial`, and the lit scene
> is then graded by the `VisualTreatment` into MW's final look.

[← C57.3 — Weather & rain](03-weather-rain.md) · [Chapter 57 hub](C57-World-Systems.md) ·
[Next: C57.5 — Reading world systems in RE →](05-reading-world.md)

---

## The lighting inputs

The atmosphere systems ([C57.1](01-time-of-day.md)–[C57.3](03-weather-rain.md)) are *inputs* to the **world
lighting** — the illumination every surface is drawn in ([C51.3](../C51-Render-Pipeline/03-effect-system.md)):

- **The sun (`SunColor`, [C57.1](01-time-of-day.md))** — the **key light**: the primary directional light, warm and
  low at the golden hour, casting the shadows and highlights.
- **The sky (`SkyColor`, [C57.2](02-sky-fog.md))** — the **ambient/fill**: the sky's colour lights the shadowed
  side of things (skylight), so shadows aren't black but sky-tinted.
- **The fog (`FogValue`, [C57.2](02-sky-fog.md))** — the **atmosphere**: fading distant surfaces into the haze.
- **The weather ([C57.3](03-weather-rain.md))** — modifies the above (rain darkens, wets, dulls).

So the lighting is composed from the atmosphere: a warm key (sun) + a tinted fill (sky) + a haze (fog), all set by
the time and weather. This is *physically-motivated* lighting — the sun and sky as real light sources — which is
why the world feels naturally lit rather than flatly coloured.

> ✅ *Verified:* `SunColor` ([C57.1](01-time-of-day.md)), `SkyColor` ([C57.2](02-sky-fog.md)), `FogValue`
> ([C57.2](02-sky-fog.md)), and `LightMaterial` are present in `speed.exe` — the lighting inputs and the surface
> lighting.

## LightMaterial: applying light to surfaces

The **`LightMaterial`** (verified) is where the lighting *meets the surfaces* — the material lighting model
([C51.3](../C51-Render-Pipeline/03-effect-system.md)) that combines the light inputs with each surface's material:

- **The surface's material** ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) — its colour,
  reflectivity, textures.
- **The light inputs** — the sun (key), sky (fill), and atmosphere (fog).
- **→ the lit pixel** — the surface's final shaded colour, via the material's shader
  ([C51.3](../C51-Render-Pipeline/03-effect-system.md)).

So `LightMaterial` is the *shading* step ([C51.3](../C51-Render-Pipeline/03-effect-system.md)) that applies the
world lighting to each surface — a car's paint lit by the warm sun and sky-tinted shadow, an asphalt road lit and
fogged. Every surface in the scene is drawn through this — the world lighting × the surface material = the lit
scene. This is where the *atmosphere* (the light) becomes *pixels* (the lit surfaces).

## The chain to the final look

The full chain from atmosphere to the final frame is now complete
([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)):

```
TimeOfDay (sun) + Sky + Weather + Fog   ← the atmosphere (C57.1-3)
   → world lighting (key + fill + haze)
      → LightMaterial: light × surface material → the lit scene   ← rendered (C51.3)
         → VisualTreatment: grade the image (sepia, bloom, blur)  ← post-process (C51.4)
            → the final frame
```

So MW's look is *two stages*: the **atmosphere lights the scene** (the sun/sky/fog, applied via `LightMaterial`),
and the **treatment grades the image** (the `VisualTreatment`, [C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).
The golden-hour warmth comes from *both* — the warm sun *lighting* the scene, and the warm grade *over* it,
compounding into the signature look. This is why MW's world is so unmistakably warm: it's lit warm *and* graded
warm. Understanding the look means understanding this two-stage chain — atmosphere (lighting) then treatment
(grading), the world systems feeding the renderer feeding the treatment.

## Why physically-motivated lighting

Lighting the world from *atmosphere inputs* (sun, sky, fog) rather than flat baked lighting serves realism and
consistency:

- **It's coherent.** Every surface is lit by the *same* sun and sky, so the world is *consistently* lit — a car and
  the road it's on share the golden-hour light, so they look like they belong together.
- **It's dynamic.** Because the light is the sun's *state* ([C57.1](01-time-of-day.md)), shadows and highlights are
  *dynamic* (they track geometry and view) — the world feels three-dimensional and alive.
- **It composes with the treatment.** The physically-lit scene ([C51.3](../C51-Render-Pipeline/03-effect-system.md))
  is a *neutral-ish* base that the `VisualTreatment` ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) then
  stylises — a clean separation of *lighting* (physical) and *grading* (stylistic).

So world lighting is the *physical* half of MW's look, and the visual treatment is the *stylistic* half — together,
a lit-then-graded world. This two-stage design ([above](#the-chain-to-the-final-look)) is a mature rendering
architecture: light the scene realistically from world state, then grade the image for mood. The atmosphere systems
([Chapter 57](C57-World-Systems.md)) are the *world state* that lights it; understanding them completes the picture
of how Rockport comes to look the way it does.

## RE implications

- **Lighting inputs** — sun (`SunColor`, key), sky (`SkyColor`, fill), fog (`FogValue`, atmosphere), weather
  (modifier).
- **`LightMaterial`** applies the light to surfaces — light × material = the lit scene
  ([C51.3](../C51-Render-Pipeline/03-effect-system.md)).
- **Two-stage look** — atmosphere lights the scene, `VisualTreatment` grades the image — both warm, compounding.
- **Physically-motivated lighting** is coherent, dynamic, and composes cleanly with the stylistic treatment.

---

### Key takeaways

- The atmosphere systems are the **lighting inputs** — the sun (`SunColor`, **key light**), the sky (`SkyColor`,
  **ambient fill**), and the fog (`FogValue`, **atmosphere**), modified by weather.
- **`LightMaterial`** applies the light to each surface — **light × surface material = the lit scene**
  ([C51.3](../C51-Render-Pipeline/03-effect-system.md)) — where atmosphere becomes pixels.
- MW's look is **two stages**: the atmosphere **lights** the scene (warm sun/sky via `LightMaterial`), then the
  **`VisualTreatment` grades** the image (warm sepia/bloom) — **lit warm *and* graded warm**.
- **Physically-motivated lighting** (from sun/sky/fog) is **coherent** (everything shares the light), **dynamic**
  (shadows/highlights track geometry), and **composes** with the stylistic treatment.
- The atmosphere systems are the **world state** that lights Rockport — understanding them completes the picture of
  MW's unmistakable look.

**Continue:** [C57.5 — Reading world systems in RE](05-reading-world.md) · [Chapter 57 hub](C57-World-Systems.md)
