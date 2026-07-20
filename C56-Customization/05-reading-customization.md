# C56.5 — Reading Customization in RE

> **The one-sentence version:** navigate customization by the `Customize*` screens, the `PART_*` categories and
> `PerformanceLevel`/`PerformanceMatching`, the `TuningSlider`s, and the visual data (`Paint`/`Vinyls`/`Rims`) —
> reading it all as a garage front-end onto the car's vault data.

[← C56.4 — Visual customization](04-visual.md) · [Chapter 56 hub](C56-Customization.md) ·
[Next: Chapter 57 — World Systems: Time, Weather & Lighting →](../C57-World-Systems/C57-World-Systems.md)

---

## Anchors for customization RE

The customization system is anchored on verified UI and data strings:

- **The `Customize*` screens** — `CustomizeParts`, `CustomizePaint`, `CustomizeDecals`, `CustomizeNumbers`
  ([C56.1](01-two-customizations.md)).
- **The `PART_*` categories** — engine/suspension/brake/tire/trans/turbo ([C56.2](02-performance-parts.md)).
- **`PerformanceLevel`/`PerformanceMatching`** — the tiers and scaling ([C56.2](02-performance-parts.md)).
- **`TuningScreen`/`TuningSlider`** — the fine-tuning ([C56.3](03-tuning-sliders.md)).
- **The visual data** — `Paint`/`PaintDatum`, `Vinyls`, `Decals`, `Rims`, `Spoilers`, `bodykit`
  ([C56.4](04-visual.md)).

From these, customization is navigable: the UI, the performance parts, the tuning, and the visual layers.

## The RE workflow

Reading customization:

1. **Map the `Customize*` UI** — the screens and categories ([C56.1](01-two-customizations.md)).
2. **Trace parts to the vault** — how `PART_*` installation edits the car's tuning fields
   ([C56.2](02-performance-parts.md), [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
3. **Follow the tuning sliders** — the fine-tuning fields ([C56.3](03-tuning-sliders.md)).
4. **Map the visual layers** — paint/decal data and mesh selection ([C56.4](04-visual.md)).

The output is the full customization picture: the UI, and how it edits the car's vault data.

## Customization is the vault editor

The unifying RE insight: **customization is a vault editor** ([C56.1](01-two-customizations.md)). Everything the
garage does reduces to *editing the car's vault data* ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)):

- **A part** ([C56.2](02-performance-parts.md)) writes a parameter set into the car's tuning fields.
- **A slider** ([C56.3](03-tuning-sliders.md)) sweeps a handling field continuously.
- **A paint/decal/kit** ([C56.4](04-visual.md)) sets a visual data value or mesh selection.

So to understand customization, you understand *which vault fields each control edits*
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). The garage UI is a friendly front-end onto the
reflection-hashed vault ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) — the player editing the
same fields the developers tune ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)). This means a
*mod* that adds parts or tuning is a *vault edit* ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)),
not a code change — the customization system applies any data. Reading customization is thus reading the *player's
interface to the vault* — the point where the data-driven architecture becomes user-facing.

## Customization closes the car loop

With customization decoded, the *car* is fully understood end to end — from data, through simulation, to the
garage:

- **The car is vault data** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — its parameters and
  assets.
- **The sim reads it** ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) — the
  driving model from the numbers.
- **The renderer draws it** ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) — the visuals from the
  assets.
- **Customization edits it** (this chapter) — the player authoring the data.

So the car is a *complete loop*: data defines it, the sim/renderer express it, and customization lets the player
*author* it. This is the fullest expression of the data-over-code architecture
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) — the same vault data
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) that the developers author, the sim consumes, the
renderer draws, and *the player edits*. Reading the car across these chapters
([13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md), [42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md),
[51](../C51-Render-Pipeline/C51-Render-Pipeline.md), 56) shows the whole lifecycle of a car as *data* — the central
object of the game, understood completely.

## RE implications

- **Anchor on** the `Customize*` screens, the `PART_*` categories, `PerformanceLevel`/`Matching`, the sliders, and
  the visual data.
- **The RE workflow** — map the UI → trace parts to the vault → follow the sliders → map the visual layers.
- **Customization is a vault editor** — every control edits the car's vault data; a mod is a vault edit.
- **It closes the car loop** — data defines, sim/renderer express, customization authors — the full data-over-code
  lifecycle.

---

### Key takeaways

- Customization is anchored on the **`Customize*` screens**, the **`PART_*` categories**, **`PerformanceLevel`/
  `PerformanceMatching`**, the **`TuningSlider`s**, and the **visual data** (`Paint`/`Vinyls`/`Rims`).
- The RE workflow: **map the UI → trace parts to the vault → follow the sliders → map the visual layers**.
- The core insight is that **customization is a vault editor** — every control edits the car's vault data
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); a **mod is a vault edit**, not code.
- Customization **closes the car loop** — data **defines** the car, the sim/renderer **express** it, and
  customization lets the player **author** it — the fullest data-over-code lifecycle.
- Read across Chapters 13, 42, 51, and 56, the **car is understood completely** as data — the central object of the
  game.

**Next:** [Chapter 57 — World Systems: Time, Weather & Lighting](../C57-World-Systems/C57-World-Systems.md): the
world the cars inhabit.

**Sources:** `speed.exe` (verified: `Customize*` screens — `CustomizeParts`/`CustomizePaint`/`CustomizePaintDatum`/
`CustomizeDecals`/`CustomizeNumbers`/`CustomizeHUDColor`/`CustomizeCategory`/`CustomizePartOption`; `PART_ENGINE`/
`PART_SUSPENSION`/`PART_BRAKE`/`PART_TIRE`/`PART_TRANS`/`PART_TURBO`; `PerformanceLevel`/`PerformanceMatching`;
`TuningScreen`/`TuningSlider`; `Paint`/`PaintDatum`/`Vinyls`/`Decals`/`Rims`/`Spoilers`/`bodykit`; `Marker`).
