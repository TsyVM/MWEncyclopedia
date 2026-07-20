# Chapter 57 — World Systems: Time, Weather & Lighting

> **Goal of this chapter:** decode the atmospheric systems that give Rockport its world — the time-of-day
> (`TimeOfDay`, `SunRise`/`Sunset`, `SunColor`), the sky (`skybox`, `SkyColor`, `skyshader`), the weather
> (`Weather`, `Rain`/`RainDropShader`), and the fog (`FogShader`/`FogValue`) that together set the mood the
> `VisualTreatment` grades.

Rockport isn't a neutral backdrop — it has a *time*, a *sky*, and an *atmosphere* that make it feel like a real
place (Most Wanted's signature perpetual golden-hour). This chapter decodes the world-atmosphere systems: the
time-of-day and sun that light it, the skybox and fog that frame it, and the weather (mostly dry, with rain
support) that varies it. These feed the lighting and the `VisualTreatment`
([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) — the systems that give the world its unmistakable look.

> **Verified against the executable.** The atmosphere systems are named in `speed.exe`: **`TimeOfDay`** with
> `SunRise`, `Sunset`, `SunColor` (the sun cycle); the **sky** — `skybox`, `SkyColor`, `SkyBlendFactor`,
> `SkyTextureMisc`, `skyshader`, `skyrgba`; **weather** — `Weather`, `WeatherReport`, and `Rain`/`RainEnable`/
> `RainDropShader` (`raindropalpha`/`raindropoffset`); **fog** — `FogShader`, `FogValue`; and **`LightMaterial`**
> for surface lighting. These feed the state-driven `VisualTreatment`
> ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).

---

## Deep-dive pages

- [C57.1 — Time of day & the sun](01-time-of-day.md): `TimeOfDay`, the sun cycle, and MW's golden hour.
- [C57.2 — Sky & fog](02-sky-fog.md): the skybox, sky colour, and atmospheric fog.
- [C57.3 — Weather & rain](03-weather-rain.md): the mostly-dry world with rain support.
- [C57.4 — World lighting](04-world-lighting.md): how time/sky/weather feed lighting and the treatment.
- [C57.5 — Reading world systems in RE](05-reading-world.md): navigating the atmosphere.

---

## 57.1 Time of day & the sun

The world has a **`TimeOfDay`** ([C57.1](01-time-of-day.md)) — a sun cycle (`SunRise`, `Sunset`, `SunColor`) that
sets where the sun is and what colour it casts. Most Wanted is famous for its **perpetual late-afternoon** — a warm,
low, golden-hour sun that defines its look. The sun's position and colour drive the world lighting
([C57.4](04-world-lighting.md)) and the shadows, and the warm sun is a big part of what the `VisualTreatment`
([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) leans into.

## 57.2 Sky & fog

The world is framed by a **skybox** ([C57.2](02-sky-fog.md)) — `SkyColor`, `SkyBlendFactor`, `skyshader`, `skyrgba`
— the distant sky drawn behind everything, and **fog** (`FogShader`, `FogValue`) — the atmospheric haze that fades
distant geometry ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)). The sky sets the *backdrop* colour/mood;
the fog sets the *depth* atmosphere (and hides the far draw distance,
[Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)). Together they build the world's sense of *space and
air*.

## 57.3 Weather & rain

Rockport is **mostly dry** ([C57.3](03-weather-rain.md)) — Most Wanted's world is predominantly the sunny golden
hour — but the engine supports **weather** (`Weather`, `WeatherReport`) and specifically **rain** (`Rain`,
`RainEnable`, `RainDropShader`, `raindropalpha`/`raindropoffset`). Rain adds raindrops on the screen/camera
([C52.1](../C52-Effects-Particles/01-two-worlds.md)) and wet-road effects. That rain is *supported* but *rare*
reflects a deliberate mood choice — the game's identity is the warm, dry chase, with weather as an occasional
variation.

## 57.4 World lighting

Time, sky, and weather all feed the **world lighting** ([C57.4](04-world-lighting.md)): the `SunColor` sets the key
light, the sky sets the ambient, the fog sets the atmosphere, and `LightMaterial` applies them to surfaces
([C51.3](../C51-Render-Pipeline/03-effect-system.md)). The lit scene is then graded by the `VisualTreatment`
([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) — so the world's *look* is the atmosphere systems
(lighting the scene) plus the treatment (grading the image). This is the full chain from "what time/weather it is"
to "how the frame looks."

---

### Key takeaways

- The world has a **`TimeOfDay`** — a sun cycle (`SunRise`/`Sunset`/`SunColor`) — famous for MW's **perpetual
  golden hour** (warm, low sun) that defines its look.
- It's framed by a **skybox** (`SkyColor`/`SkyBlendFactor`/`skyshader`) and **fog** (`FogShader`/`FogValue`) — the
  backdrop and the atmospheric depth.
- Rockport is **mostly dry**, but the engine **supports weather** — `Weather`/`WeatherReport` and **rain**
  (`RainDropShader`/`RainEnable`) — an occasional mood variation.
- Time, sky, and weather feed the **world lighting** (`SunColor` → `LightMaterial`), and the lit scene is graded by
  the **`VisualTreatment`** ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).
- The chain from **"what time/weather it is" to "how the frame looks"** is the atmosphere systems plus the
  treatment.

**Next:** [Chapter 58 — The Build: Toolchain, Bundles & Asset Pipeline](../C58-Build-Pipeline/C58-Build-Pipeline.md):
how the game was made.
