# C57.1 — Time of Day & the Sun

> **The one-sentence version:** the world has a `TimeOfDay` with a sun cycle (`SunRise`, `Sunset`, `SunColor`) that
> sets the sun's position and colour — and Most Wanted's signature is its perpetual late-afternoon golden hour, a
> warm, low sun that defines the game's look.

[← Chapter 57 hub](C57-World-Systems.md) · [Next: C57.2 — Sky & fog →](02-sky-fog.md)

---

## The sun cycle

The world has a **`TimeOfDay`** system — a notion of *when it is*, with a sun that has a position and a colour. The
verified components:

- **`SunRise` / `Sunset`** — the endpoints of the day cycle; the sun's low-angle, warm-coloured moments.
- **`SunColor`** — the colour the sun casts, which shifts with time (warm at the golden hours, whiter at midday).
- **The sun's position** — where in the sky it sits, setting the light direction and the shadows.

So `TimeOfDay` drives the *primary light* of the world — the sun — in position and colour. This is the dominant
influence on how the world *looks*: the direction of the light, the length and angle of shadows, and the warmth of
the colour all come from the sun's state.

> ✅ *Verified:* `TimeOfDay`, `SunRise`, `Sunset`, and `SunColor` are present in `speed.exe` — the time-of-day sun
> cycle.

## The perpetual golden hour

Most Wanted's defining atmospheric choice is its **perpetual late-afternoon** — the game world sits in an eternal
golden hour, a warm, low, hazy sun:

- **A low sun** — long shadows, raking light across the buildings and roads, a dramatic directional key light.
- **A warm `SunColor`** — the amber/gold cast that gives everything its sun-baked warmth
  ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).
- **A consistent mood** — because the time is (largely) fixed at this golden hour, the world's lighting is *always*
  this dramatic warmth, a constant identity.

This is a deliberate art direction: rather than a full day/night cycle, MW fixes the world at its most *cinematic*
time — the golden hour photographers prize. It's a huge part of why the game *looks* the way it does, and it pairs
with the `VisualTreatment` ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) — the warm sun provides the
*lighting*, the treatment provides the *grade*, and together they create the signature sepia-warm world. The
`TimeOfDay` machinery *supports* a cycle (`SunRise`/`Sunset` exist), but the game *uses* mostly the golden hour.

> 🟡 *Reasoned:* that MW fixes the world at a perpetual golden hour (rather than running a full day/night cycle) is
> the game's documented art direction, consistent with the verified `TimeOfDay`/`SunColor` system and the warm
> visual identity; the exact time-of-day usage is per-world data. The sun-cycle system is verified.

## The sun drives lighting

The `TimeOfDay` sun is the *input* to the world lighting ([C57.4](04-world-lighting.md)):

- **Key light** — the sun is the scene's primary directional light; its `SunColor` and direction light every
  surface ([C51.3](../C51-Render-Pipeline/03-effect-system.md), `LightMaterial`).
- **Shadows** — the sun's position casts the world's shadows (long, at the golden hour).
- **Specular/highlights** — the sun's reflection off cars ([C51.3](../C51-Render-Pipeline/03-effect-system.md)) and
  wet roads is the bright, blooming highlights ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).

So the sun is the *source* from which the world's lighting flows — change the sun (time, colour), and the whole
world relights. This is why `TimeOfDay` is a *world system* and not just a cosmetic setting: it sets the light that
everything is drawn in. The chain is `TimeOfDay` (the sun) → world lighting ([C57.4](04-world-lighting.md)) →
`VisualTreatment` grade ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) → the frame. The golden-hour sun is
the first link — the *reason* the world is warm before the treatment even grades it.

## Why time of day matters

Having a `TimeOfDay` system (even used mostly at one time) matters for the world's feel:

- **It makes the world *lit*, not *painted*.** The sun is a real directional light, so the world has *dynamic*
  lighting (shadows move with geometry, highlights track the view) — not baked-in flat lighting. This makes it feel
  three-dimensional and real.
- **It anchors the mood.** The golden hour is *the* mood of Most Wanted — warm, urgent, cinematic. `TimeOfDay`
  fixing it there is what makes the mood *consistent*.
- **It supports variation.** The cycle machinery (`SunRise`/`Sunset`) means the engine *can* vary time — for
  specific events or moments — even if the main game is golden hour.

So `TimeOfDay` is the foundation of the world's atmosphere — the sun that lights it and the golden-hour mood that
defines it. It's the first of the atmosphere systems ([Chapter 57](C57-World-Systems.md)) because it's the *primary
light*; the sky ([C57.2](02-sky-fog.md)), fog ([C57.2](02-sky-fog.md)), and weather
([C57.3](03-weather-rain.md)) build on it. Understanding the world's look starts with the golden-hour sun.

## RE implications

- **`TimeOfDay`** drives a sun cycle — `SunRise`/`Sunset`/`SunColor` — the world's primary light.
- **The perpetual golden hour** is MW's signature — a warm, low sun defining the look
  ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)).
- **The sun drives lighting** — key light, shadows, highlights — everything is drawn in its light.
- **`TimeOfDay` makes the world lit** (dynamic, 3D), anchors the mood, and supports variation.

---

### Key takeaways

- The world has a **`TimeOfDay`** — a sun cycle (`SunRise`/`Sunset`/`SunColor`) setting the sun's **position and
  colour**, the world's primary light.
- Most Wanted's signature is the **perpetual golden hour** — a warm, low, hazy sun (long shadows, amber cast) — a
  deliberate art direction fixing the world at its most cinematic time.
- The sun is the **input to world lighting** — key light, shadows, and blooming highlights all flow from its state.
- `TimeOfDay` makes the world **lit, not painted** (dynamic 3D lighting), **anchors the mood** (consistent golden
  hour), and **supports variation** (the cycle exists).
- The golden-hour sun is the **first link** in the look — warm *before* the `VisualTreatment`
  ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) even grades it.

**Continue:** [C57.2 — Sky & fog](02-sky-fog.md) · [Chapter 57 hub](C57-World-Systems.md)
