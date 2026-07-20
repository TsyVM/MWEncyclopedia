# C57.5 — Reading World Systems in RE

> **The one-sentence version:** navigate the atmosphere by `TimeOfDay`/`SunColor` (the sun), the sky system
> (`skybox`/`SkyColor`/`skyshader`), the fog (`FogShader`/`FogValue`), and the weather (`Weather`/`Rain`) — reading
> the world state that lights the scene the `VisualTreatment` then grades.

[← C57.4 — World lighting](04-world-lighting.md) · [Chapter 57 hub](C57-World-Systems.md) ·
[Next: Chapter 58 — The Build: Toolchain, Bundles & Asset Pipeline →](../C58-Build-Pipeline/C58-Build-Pipeline.md)

---

## Anchors for world-systems RE

The atmosphere systems are anchored on verified strings:

- **Time of day** — `TimeOfDay`, `SunRise`, `Sunset`, `SunColor` ([C57.1](01-time-of-day.md)).
- **Sky** — `skybox`, `SkyboxCurrentGen`, `SkyColor`, `SkyBlendFactor`, `skyshader`, `skyrgba`
  ([C57.2](02-sky-fog.md)).
- **Fog** — `FogShader`, `FogValue` ([C57.2](02-sky-fog.md)).
- **Weather** — `Weather`, `WeatherReport`, `Rain`, `RainEnable`, `RainDropShader` ([C57.3](03-weather-rain.md)).
- **Lighting** — `LightMaterial` ([C57.4](04-world-lighting.md)).

From these, the atmosphere is navigable: the sun, sky, fog, weather, and their lighting.

## The RE workflow

Reading the world systems:

1. **Trace `TimeOfDay`** — the sun cycle and `SunColor` ([C57.1](01-time-of-day.md)); the primary light.
2. **Map the sky/fog** — the skybox and fog shaders ([C57.2](02-sky-fog.md)); the backdrop and atmosphere.
3. **Check the weather** — `Weather`/`Rain` ([C57.3](03-weather-rain.md)); the variation support.
4. **Follow to lighting** — how `SunColor`/`SkyColor`/`FogValue` feed `LightMaterial`
   ([C57.4](04-world-lighting.md)) and the treatment ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).

The output is the full atmosphere picture: the world state and how it lights the scene.

## The look is lit-then-graded

The key insight for the world's appearance ([C57.4](04-world-lighting.md)): MW's look is **lit-then-graded** — the
atmosphere systems *light* the scene, and the `VisualTreatment` ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md))
*grades* the image. This two-stage separation is the thing to carry away:

- **Lighting is physical** — sun, sky, fog as real light sources (this chapter), applied via `LightMaterial`
  ([C57.4](04-world-lighting.md)).
- **Grading is stylistic** — the `VisualTreatment` ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) sepia/bloom/vignette
  over the lit image.

Recognising this separation resolves a common confusion — "is MW's warmth the lighting or the filter?" The answer is
*both*, in two stages: warm light *and* warm grade. Reading the world systems (the lighting) alongside the visual
treatment (the grade) shows the complete, two-stage recipe for MW's signature look — neither alone, but their
composition.

## World systems complete the world

With the atmosphere decoded, the **world** is fully understood — geometry, streaming, and now atmosphere:

- **The world's *structure*** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)–[16](../C16-Scenery-Cull/C16-Scenery-Cull.md))
  — the streamed geometry, the roads, the scenery.
- **The world's *atmosphere*** (this chapter) — the time, sky, weather, and lighting that give it life.

Together they are Rockport — a *structured* place (buildings, roads, [Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md))
with an *atmosphere* (golden-hour light, [C57.1](01-time-of-day.md)). The structure is *what's there*; the
atmosphere is *how it feels*. A city without atmosphere is a set of models; atmosphere without structure is empty
light. Rockport is both — the streamed world lit by the golden-hour sun, framed by the warm sky and fog, that makes
it feel like a real, cinematic place. Reading the world systems completes the world: not just its geometry, but its
soul.

## RE implications

- **Anchor on** `TimeOfDay`/`SunColor`, the sky/fog shaders, the weather system, and `LightMaterial`.
- **The RE workflow** — trace `TimeOfDay` → map sky/fog → check weather → follow to lighting.
- **The look is lit-then-graded** — atmosphere lights the scene, `VisualTreatment` grades the image — both warm.
- **World systems complete the world** — structure (geometry) + atmosphere (light) = Rockport.

---

### Key takeaways

- The atmosphere is anchored on **`TimeOfDay`/`SunColor`** (the sun), the **sky system** (`skybox`/`SkyColor`/
  `skyshader`), the **fog** (`FogShader`/`FogValue`), the **weather** (`Weather`/`Rain`), and **`LightMaterial`**.
- The RE workflow: **trace `TimeOfDay` → map the sky/fog → check the weather → follow to lighting**.
- The key insight is **lit-then-graded** — the atmosphere systems **light** the scene (physical), the
  **`VisualTreatment` grades** the image (stylistic) — MW's warmth is **both**, in two stages.
- **World systems complete the world** — its **structure** (geometry/streaming,
  [Chapters 15](../C15-Track-Streaming/C15-Track-Streaming.md)–16) plus its **atmosphere** (time/sky/weather/light)
  = Rockport.
- The structure is **what's there**; the atmosphere is **how it feels** — together, a real, cinematic place.

**Next:** [Chapter 58 — The Build: Toolchain, Bundles & Asset Pipeline](../C58-Build-Pipeline/C58-Build-Pipeline.md):
how the game was made.

**Sources:** `speed.exe` (verified: `TimeOfDay`/`SunRise`/`Sunset`/`SunColor`; `skybox`/`SkyboxCurrentGen`/
`SkyColor`/`SkyBlendFactor`/`SkyTextureMisc`/`skyshader`/`skyrgba`; `FogShader`/`FogValue`; `Weather`/`WeatherReport`/
`Rain`/`RainEnable`/`RainDropShader`/`raindropalpha`/`raindropoffset`; `LightMaterial`).
