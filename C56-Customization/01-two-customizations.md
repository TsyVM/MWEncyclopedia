# C56.1 — The Two Customizations

> **The one-sentence version:** customization splits into performance (functional — makes the car faster and
> handle better) and visual (cosmetic — makes it distinctive), each a `Customize*` screen, and both are front-ends
> onto the car's vault data.

[← Chapter 56 hub](C56-Customization.md) · [Next: C56.2 — Performance parts →](02-performance-parts.md)

---

## Two axes of customization

Building a car in Most Wanted happens along **two axes**, and the `Customize*` screens
([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) cover both:

- **Performance** (`CustomizeParts`, [C56.2](02-performance-parts.md)) — the *functional* upgrades: engine,
  suspension, brakes, tyres, etc. These make the car **faster and handle better** — they change how it *drives*.
- **Visual** (`CustomizePaint`, `CustomizeDecals`, [C56.4](04-visual.md)) — the *cosmetic* choices: paint, vinyls,
  body kits, rims. These make the car **distinctive** — they change how it *looks*.

So one axis is *substance* (go faster) and the other is *style* (look cool). Both matter in Most Wanted — you tune
performance to win races ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) and pursuits
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), *and* you style your car to make it yours. The `Customize*`
UI family ([verified](#re-implications)) presents both as garage categories.

> ✅ *Verified:* the customization UI is a `Customize*` family — `CustomizeParts`, `CustomizePaint`/`PaintDatum`,
> `CustomizeDecals`, `CustomizeNumbers`, `CustomizeHUDColor`, `CustomizeCategory`, `CustomizePartOption`,
> `CustomizeMain` — present in `speed.exe`.

## Both edit the vault

The unifying fact ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)): *both* customizations are
**front-ends onto the car's vault data** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)):

- **Performance customization edits the tuning *numbers*.** Installing an engine part
  ([C56.2](02-performance-parts.md)) raises the car's power-curve vault fields
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)); a tuning slider
  ([C56.3](03-tuning-sliders.md)) adjusts a handling field. The sim
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) then reads the new numbers.
- **Visual customization edits the *assets/selection*.** Choosing paint ([C56.4](04-visual.md)) sets the car's
  paint colour; a body kit selects a different mesh ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)). The
  renderer ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) then draws the new look.

So customization is the *editor* for a car's data — performance edits the *simulation* data (numbers), visual edits
the *presentation* data (assets). This is why the whole engine is data-driven
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)): so that customization can *change the car* by
editing data, without touching code. The garage UI is a friendly front-end onto the vault
([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) — a way for the player to author the car's data.

## Customization is the payoff of data-over-code

The customization system is the *reason* — the payoff — for the data-over-code architecture seen throughout the
book ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)):

- **The engine reads data** — every car system (engine, suspension, tyres, damage, paint,
  [Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) is parameterised by vault
  data, not hard-coded.
- **So customization can rewrite it** — because the car *is* its data, changing the data changes the car. A player
  installing parts is *rewriting the car's vault*, and the sim/render just reads the new values.
- **No code per customization** — a new part is new *data* (a parameter set), not new code. The engine's part
  system ([C56.2](02-performance-parts.md)) applies any part by editing the fields it names.

So customization is where the data-over-code investment pays off: the entire pipeline (formats
[Part I](../C6-Texture-Codecs/C6-Texture-Codecs.md), vault [Part II](../C11-Attribute-Vaults/C11-Attribute-Vaults.md),
sim [Part VIII](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) is built to be *data-driven* precisely so that
*the player can customize*. The garage is the user-facing expression of the whole architecture — the point at which
"the car is data" becomes "you build the car." Every earlier chapter's data-over-code note
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) leads here.

## Why split performance and visual

Separating performance and visual customization is clean design:

- **Different data, different UI.** Performance edits numbers (parts, sliders,
  [C56.2](02-performance-parts.md)–[C56.3](03-tuning-sliders.md)); visual edits assets (paint, decals,
  [C56.4](04-visual.md)). Each gets a UI suited to its data.
- **Different purposes.** Performance is for *winning* (functional); visual is for *identity* (expressive) — players
  engage with each differently.
- **Independent progression.** You can upgrade performance without changing looks, or restyle without re-tuning —
  the two axes are orthogonal.

So the split reflects the two *reasons* to customize — to go faster and to look distinctive — each with its own
data and UI. Together they let you build a car that's *both* fast *and* yours: the performance to climb the
Blacklist ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) and the style to make the climb feel
personal. The next pages take each axis in turn — performance ([C56.2](02-performance-parts.md)–[C56.3](03-tuning-sliders.md))
and visual ([C56.4](04-visual.md)).

## RE implications

- **Two customization axes** — performance (functional) and visual (cosmetic) — each a `Customize*` screen.
- **Both edit the vault** — performance edits tuning *numbers*, visual edits *assets/selection*.
- **Customization is the payoff of data-over-code** — the car *is* its data, so editing data builds the car.
- **The split** reflects two purposes (win vs. identity), two data types, and independent progression.

---

### Key takeaways

- Customization has **two axes**: **performance** (functional — faster, better handling) and **visual** (cosmetic —
  distinctive), each a `Customize*` screen.
- **Both are front-ends onto the vault** — performance edits the tuning **numbers**
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) the sim reads; visual edits the **assets/selection**
  the renderer draws.
- Customization is the **payoff of data-over-code** — because the car *is* its data, editing data **builds the
  car**, with no code per part.
- The garage UI is the **user-facing expression** of the whole data-driven architecture — where "the car is data"
  becomes "you build the car."
- The **split** (performance vs. visual) reflects two purposes (winning vs. identity), two data types, and
  orthogonal progression.

**Continue:** [C56.2 — Performance parts](02-performance-parts.md) · [Chapter 56 hub](C56-Customization.md)
