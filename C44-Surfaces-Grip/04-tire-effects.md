# C44.4 — TireEffectRecord: the Visual

> **The one-sentence version:** `TireEffectRecord` (verified vault key, ×50 in `attributes.bin`) maps a (surface ×
> tyre mode) to the right `fxtd_*` terrain-driving effect — so a drift on asphalt smokes differently than rolling
> on sand, because the mode crossed with the surface picks the exact effect.

[← C44.3 — RoadNoiseRecord](03-road-noise.md) · [Chapter 44 hub](C44-Surfaces-Grip.md) ·
[Next: C44.5 — The three synchronized reads →](05-three-reads.md)

---

## The visual read

The **visual** read of the surface is governed by **`TireEffectRecord`** — a vault record that maps a *(surface ×
tyre mode)* pair to the terrain-driving particle effect (the smoke, dust, and debris the tyres kick up). Verified,
its reflection hash `0x681D219C` appears **50 times** in `attributes.bin` — the most of the three surface records
([C44.3](03-road-noise.md)), because it's keyed on *two* dimensions (surface *and* mode), so it needs the most
entries.

The effect it selects is a **`fxtd_*`** effect (`fx` = effect, `td` = terrain-drive), named by mode and surface:
`fxtd_<mode>_<surface>`.

> ✅ *Verified:* `rh("TireEffectRecord")=0x681D219C` appears **×50** as a vault key in `GLOBAL/attributes.bin` — the
> tyre-effect record per surface × mode. The `fxtd_*` effects it selects live in the effect bank / ErtS table (not
> as literal strings in `speed.exe`).

## The tyre mode

The key second dimension is the **tyre mode** — *what the tyre is doing*
([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)):

| Mode | `fxtd_` prefix | What the tyre is doing |
|---|---|---|
| driving | `fxtd_dr_*` | rolling normally |
| skid | `fxtd_sk_*` | braking / locked |
| slide | `fxtd_sl_*` | drifting |
| fly | `fxtd_fly_*` | leaving the ground |
| hit | `fxtd_hit_*` | landing |

The mode is a readout of the tyre's slip state ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) —
the tyre model produces it, and `TireEffectRecord` consumes it. So the visual effect reflects not just *where* the
tyre is but *how* it's being driven.

## Surface × mode = the exact effect

Crossing the mode with the surface ([C44.1](01-surface-taxonomy.md)) gives the exact effect
([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)):

- **A drift on asphalt** → `fxtd_sl_asphalt` — white tyre smoke.
- **Rolling on sand** → `fxtd_dr_sand` — a low dust plume.
- **A braking skid on gravel** → `fxtd_sk_gravel` — flying stones and dust.
- **Landing on grass** → `fxtd_hit_grass` — a scatter of turf.

This two-dimensional keying is what makes the visuals *specific*: the game doesn't just show "some smoke" — it
shows the smoke that *this surface* makes when the tyre is doing *this thing*. A drift (slide) on asphalt is white
smoke; the same drift on dirt is a brown dust cloud. The `fxtd_*` bank ([Chapter 19 in the archive's terms; here
the effects system]) exists *because* `TireEffectRecord` needs one entry per surface × mode — its ×50 references
are that grid filled in.

## Why the visual completes the trio

`TireEffectRecord` is the **look** of the surface, completing the trio with grip (feel, [C44.2](02-grip.md)) and
`RoadNoiseRecord` (sound, [C44.3](03-road-noise.md)):

- **You see the surface.** The dust off a dirt road, the smoke of a drift, the spray through water — the effect
  tells you what you're on and what your tyres are doing, visually.
- **Drifts read clearly.** The white smoke of a slide ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md))
  is a core visual of the racing genre — `TireEffectRecord` is what produces it, keyed to the slide mode.
- **The world looks textured.** Like the sound ([C44.3](03-road-noise.md)), the effects give each surface a visual
  signature that matches its feel and sound — coherent because all three key off the tag
  ([C44.5](05-three-reads.md)).

So the visual read is the third sense of the surface: you feel it (grip), hear it (road noise), and see it (tyre
effects). All three from one tag, and `TireEffectRecord` — the most-referenced of the three — is the richest,
because it varies with both surface and mode.

## RE implications

- **`TireEffectRecord` (×50)** is the surface's **visual** read — the `fxtd_*` smoke/dust/debris per surface ×
  mode.
- **The tyre mode** (driving/skid/slide/fly/hit) is the second key dimension — what the tyre is doing.
- **Surface × mode = the exact effect** — `fxtd_sl_asphalt` (drift smoke) vs. `fxtd_dr_sand` (dust); the grid is
  the ×50 entries.
- **The visual completes the trio** — feel (grip) + sound (road noise) + look (tyre effects), all from one tag.

---

### Key takeaways

- **`TireEffectRecord`** (verified vault key, **×50** — the most-referenced surface record) is the surface's
  **visual** read — the `fxtd_*` terrain-driving effects.
- It keys on **surface × tyre mode** (driving/skid/slide/fly/hit) — two dimensions, which is why it needs the most
  entries.
- **Surface × mode picks the exact effect**: a drift on asphalt → `fxtd_sl_asphalt` (white smoke), rolling on sand
  → `fxtd_dr_sand` (dust).
- The **drift smoke** — a core visual of the genre — is `TireEffectRecord` producing `fxtd_sl_*` for the slide
  mode.
- The visual **completes the trio** — feel (grip) + sound (road noise) + look (effects), all from one surface tag.

**Continue:** [C44.5 — The three synchronized reads](05-three-reads.md) · [Chapter 44 hub](C44-Surfaces-Grip.md)
