# C57.3 — Weather & Rain

> **The one-sentence version:** Rockport is mostly dry — the golden-hour identity — but the engine supports weather
> (`Weather`, `WeatherReport`) and specifically rain (`Rain`, `RainEnable`, `RainDropShader`), an occasional
> variation that adds screen raindrops and wet-road atmosphere.

[← C57.2 — Sky & fog](02-sky-fog.md) · [Chapter 57 hub](C57-World-Systems.md) ·
[Next: C57.4 — World lighting →](04-world-lighting.md)

---

## A mostly-dry world

Most Wanted's world is **predominantly dry** — the warm, sunny golden hour ([C57.1](01-time-of-day.md)) is the
game's identity, and most of the time it doesn't rain. But the engine has a **weather** system (`Weather`,
`WeatherReport`), and it supports **rain** specifically:

- **`Weather` / `WeatherReport`** — the weather-state system.
- **`Rain` / `RainEnable`** — the rain toggle/state.
- **`RainDropShader` / `raindropalpha` / `raindropoffset`** — the rain rendering (drops on the screen/camera).

So while the *default* is dry, the engine can *make it rain* — and rain has real rendering support (a dedicated
drop shader). That rain is *supported but rare* is the key fact: it's a variation the engine can do, deployed
occasionally (specific events, moments) for contrast, not the norm.

> ✅ *Verified:* `Weather`, `WeatherReport`, `Rain`, `RainEnable`, `RainDropShader`, `raindropalpha`, and
> `raindropoffset` are present in `speed.exe` — the weather system with rain support.

## Rain rendering: drops on the lens

The verified `RainDropShader` (with `raindropalpha`/`raindropoffset`) is a **screen-space rain effect** — raindrops
on the *camera/lens*, a post-process-like layer ([C52.1](../C52-Effects-Particles/01-two-worlds.md)):

- **`RainDropShader`** — renders drops on the screen, as if on the camera lens or windscreen.
- **`raindropalpha`** — the drops' transparency (how visible they are).
- **`raindropoffset`** — the drops' position/movement (they streak and slide with speed).

So rain in MW is rendered partly as a *screen effect* — drops on the view that streak as you drive, adding the
*feel* of driving in rain. This is the same screen-space technique as the visual treatment's blur/vignette
([C52.1](../C52-Effects-Particles/01-two-worlds.md)) — a 2D layer over the frame. Combined with wet-road *material*
changes (darker, more reflective asphalt) and reduced *grip* (wet surfaces,
[Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)), rain transforms the *look and feel* of driving — the
world glistens, the drops streak, and the car slides more.

> 🟡 *Reasoned:* the wet-road material/grip changes accompanying rain are the standard wet-weather model,
> consistent with the verified rain rendering and the surface-grip system
> ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)); the exact wet-surface parameters are per-world data.
> The rain rendering system is verified.

## Why rain is rare

That MW *supports* rain but *uses* it sparingly is a deliberate mood choice, and instructive:

- **The identity is the golden hour.** MW's mood ([C57.1](01-time-of-day.md)) — warm, sunny, urgent — is central to
  its feel. Rain would dilute that if constant; kept rare, it's a *contrast* that makes the sunny norm feel
  intentional.
- **Rain is expensive and different.** Rain rendering (drop shader, wet materials, reduced grip) is extra cost and a
  different driving feel; deploying it selectively (not everywhere) keeps the common case efficient and consistent.
- **It's a tool for specific moments.** Rare weather can mark *specific events* — a dramatic pursuit, a story beat —
  giving those moments a distinct atmosphere, precisely *because* it's unusual.

So the weather system embodies a design principle: *build the capability, deploy it for contrast*. The engine can
rain, but the game mostly doesn't — reserving weather as a variation that punctuates the golden-hour identity rather
than replacing it. This is the same "build broad, use focused" pattern as the vehicle-type breadth
([C41.6](../C41-Physics-RigidBody/06-vehicle-types.md)) — the engine's capabilities exceed what any single game
configuration uses, and the *configuration* (mostly dry) is a deliberate curation of the *capability* (weather
support).

## RE implications

- **Rockport is mostly dry** — the golden-hour identity — but the engine supports `Weather`/`WeatherReport`.
- **Rain** (`Rain`/`RainEnable`/`RainDropShader`) is supported — screen-space drops (`raindropalpha`/`offset`) plus
  wet roads.
- **Rain is a screen effect** — drops on the lens streaking with speed — like the visual treatment's 2D layers.
- **Rare by design** — weather is a *contrast* variation, reserved for moments, preserving the sunny identity.

---

### Key takeaways

- Most Wanted's world is **predominantly dry** (the golden-hour identity), but the engine **supports weather** —
  `Weather`/`WeatherReport` — and specifically **rain** (`Rain`/`RainEnable`/`RainDropShader`).
- **Rain is rendered as a screen effect** — drops on the camera/lens (`raindropalpha`/`raindropoffset`) that streak
  with speed — plus wet-road materials and reduced grip
  ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)).
- Rain **transforms the look and feel** — the world glistens, drops streak, the car slides more.
- Rain is **rare by design** — a *contrast* that makes the sunny norm feel intentional and marks special moments,
  reserving an expensive, different-feeling mode.
- The weather system is the **"build broad, use focused"** pattern — the engine's capability (weather) exceeds the
  configuration's use (mostly dry).

**Continue:** [C57.4 — World lighting](04-world-lighting.md) · [Chapter 57 hub](C57-World-Systems.md)
