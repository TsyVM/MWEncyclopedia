# Chapter 56 — Customization: Performance & Visual

> **Goal of this chapter:** decode how you build your car — the performance parts (`PART_ENGINE`/`SUSPENSION`/
> `BRAKE`/`TIRE`/`TRANS`/`TURBO` at `PerformanceLevel` tiers), the tuning sliders (`TuningScreen`/`TuningSlider`),
> and the visual customization (`Paint`, `Vinyls`, `Decals`, `Rims`, `Spoilers`, body kits) — all editing the
> vault data the sim consumes.

The cars you race ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) aren't fixed — you *build* them, in the
crib ([C54.2](../C54-GameFlow-Blacklist/02-career-manager.md)). This chapter decodes the two customization systems:
**performance** (parts and tuning that make the car *faster and handle better*) and **visual** (paint, vinyls,
kits that make it *yours*). Both are front-ends onto vault data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
— customization *edits the numbers and assets* the sim ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md))
and renderer ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) consume.

> **Verified against the executable.** The customization UI is a family of `Customize*` screens: `CustomizeParts`,
> `CustomizePaint`/`CustomizePaintDatum`, `CustomizeDecals`, `CustomizeNumbers`, `CustomizeHUDColor`,
> `CustomizeCategory`, `CustomizePartOption`. Performance parts are `PART_*` categories — `PART_ENGINE`,
> `PART_SUSPENSION`, `PART_BRAKE`, `PART_TIRE`, `PART_TRANS`, `PART_TURBO`, `PART_ECU`(-adjacent) — at
> `PerformanceLevel` tiers, with `PerformanceMatching`. Tuning is `TuningScreen`/`TuningSlider`. Visual assets are
> `Paint`/`PaintDatum`, `Vinyls`, `Decals`, `Rims`, `Spoilers`, `bodykit`. Winning a rival's car uses the `Marker`
> system.

---

## Deep-dive pages

- [C56.1 — The two customizations](01-two-customizations.md): performance vs. visual, both vault-driven.
- [C56.2 — Performance parts](02-performance-parts.md): the `PART_*` categories and `PerformanceLevel` tiers.
- [C56.3 — Tuning sliders](03-tuning-sliders.md): `TuningScreen`/`TuningSlider` fine-tuning.
- [C56.4 — Visual customization](04-visual.md): paint, vinyls, decals, kits, rims.
- [C56.5 — Reading customization in RE](05-reading-customization.md): navigating the systems.

---

## 56.1 Two customizations

Customization splits into **performance** (functional — makes the car faster/handle better) and **visual**
(cosmetic — makes it look distinctive), each a `Customize*` screen ([C56.1](01-two-customizations.md)). Both are
**front-ends onto the vault** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — performance edits the
tuning *numbers* ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)), visual edits the *assets*
(paint, decals). The customization system is thus the *editor* for the car's vault data, presented as a garage UI.

## 56.2 Performance parts

Performance is upgraded by **parts** in categories ([C56.2](02-performance-parts.md)) — `PART_ENGINE`,
`PART_SUSPENSION`, `PART_BRAKE`, `PART_TIRE`, `PART_TRANS`(mission), `PART_TURBO` — each installable at a
**`PerformanceLevel`** tier (stock → street → pro → the top tier). Installing a part *raises the relevant vault
parameters* ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) — a better engine part raises the
power curve, sport suspension raises the spring rates. `PerformanceMatching` scales events to your car's level.

## 56.3 Tuning sliders

Beyond discrete parts, **tuning sliders** ([C56.3](03-tuning-sliders.md)) — `TuningScreen`/`TuningSlider` — let you
*fine-tune* handling within the installed parts' range: brake balance, steering sensitivity, ride height, gearing.
Each slider maps to a vault field ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); moving it adjusts
that parameter continuously. Sliders are the *fine* control (continuous tuning) atop the *coarse* control (discrete
parts) — dialing in a car's exact feel.

## 56.4 Visual customization

Visual customization ([C56.4](04-visual.md)) makes the car *yours* — `Paint`/`PaintDatum` (body colour),
`Vinyls`/`Decals` (graphics), `Rims` (wheels), `Spoilers`/`bodykit` (body parts), `CustomizeNumbers` (racing
numbers), even `CustomizeHUDColor`. These edit the car's *visual* vault data / mesh selection — the renderer
([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) then draws the customised car. Unlike performance,
visual customization is *cosmetic* — it changes how the car *looks*, not how it drives (with the notable exception
that some visual parts have small aero/performance ties).

---

### Key takeaways

- Customization splits into **performance** (functional) and **visual** (cosmetic) — both **front-ends onto the
  vault** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), a garage UI editing the car's data.
- **Performance parts** — `PART_ENGINE`/`SUSPENSION`/`BRAKE`/`TIRE`/`TRANS`/`TURBO` at **`PerformanceLevel`** tiers
  — raise the sim's tuning numbers ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)).
- **Tuning sliders** (`TuningScreen`/`TuningSlider`) fine-tune handling continuously atop the discrete parts.
- **Visual customization** — `Paint`, `Vinyls`, `Decals`, `Rims`, `Spoilers`, body kits, numbers — makes the car
  distinctive, edited into its visual data for the renderer.
- Customization is the **editor for a car's vault data** — the reason the whole sim/render pipeline is data-driven
  ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)).

**Next:** [Chapter 57 — World Systems: Time, Weather & Lighting](../C57-World-Systems/C57-World-Systems.md): the
world the cars inhabit.
