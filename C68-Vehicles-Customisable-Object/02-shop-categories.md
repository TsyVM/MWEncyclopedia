# C68.2 — The Shop's Categories

> **The one-sentence version:** the customization shop is a tree of `Customize*` screens — a `CustomizeMain` root
> branching into `CustomizePerformance` and the visual categories (`CustomizePaint`, `CustomizeRims`,
> `CustomizeSpoiler`, `CustomizeDecals`, `CustomizeNumbers`, `CustomizeHUDColor`) — each category listing the parts
> that fit its slot.

[← C68.1 — The car as an object](01-car-object.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md) ·
[Next: C68.3 — Parts as catalog entries →](03-part-catalog.md)

---

## A tree of Customize screens

The shop is a **front-end** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) built from a family of `Customize*`
screens. The structure is a tree: a root, category screens, and per-part option screens.

```
CustomizeMain  (root; CustomizeMainOption per category tile)
├─ CustomizePerformance      — the functional upgrades (Chapter 69)
├─ CustomizePaint            — body colour (CustomizePaintDatum = a colour entry)
├─ CustomizeRims             — wheels
├─ CustomizeSpoiler          — rear wing
├─ CustomizeDecals           — vinyls / decals
├─ CustomizeNumbers          — racing numbers
└─ CustomizeHUDColor         — the player's HUD tint
        │
        └─ CustomizeCategory / CustomizeParts / CustomizePartOption
                                — the per-category part list and the option picker
```

`CustomizeSub` and `CustomizeGenericTop` are the shared sub-screen and header chrome; `CustomizePartOption` is the
picker that shows the choices within a slot (the five spoiler options, say). So navigating the shop is walking this
tree: pick a category on `CustomizeMain`, see its parts in `CustomizeParts`, choose one in `CustomizePartOption`.

> ✅ *Verified:* `CustomizeMain`, `CustomizeMainOption`, `CustomizeSub`, `CustomizeCategory`, `CustomizeParts`,
> `CustomizePartOption`, `CustomizeGenericTop`, `CustomizePerformance`, `CustomizePaint`, `CustomizePaintDatum`,
> `CustomizeRims`, `CustomizeSpoiler`, `CustomizeDecals`, `CustomizeNumbers`, and `CustomizeHUDColor` are all
> strings in `speed.exe` — the customization screen tree.

## Performance and visual

The categories split into the **two customizations** ([C56.1](../C56-Customization/01-two-customizations.md)):

- **Performance** — one category (`CustomizePerformance`) that fans out into the part families
  ([C68.3](03-part-catalog.md)): engine, suspension, brakes, tyres, transmission, ECU, turbo, nitrous. *Functional*
  — it changes how the car drives ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)).
- **Visual** — the rest (`CustomizePaint`, `CustomizeRims`, `CustomizeSpoiler`, `CustomizeDecals`,
  `CustomizeNumbers`, `CustomizeHUDColor`). *Cosmetic* — it changes how the car looks
  ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)).

So the shop's top level *is* the performance/visual divide, with performance as one deep category and the visual
categories as siblings. This mirrors the object's two consumers ([C68.1](01-car-object.md)): the performance
category feeds the sim (via the vault), the visual categories feed the renderer (meshes/textures/colours). The HUD
colour (`CustomizeHUDColor`) is the odd one out — it customizes *the player's on-screen UI*
([C65.4](../C65-HUD-Runtime/04-resolution-widescreen.md)) rather than the car — but it's grouped with the visual
categories because it's a cosmetic identity choice.

## A category is a slot family

Each category maps to a **slot or slot family** on the car object ([C68.1](01-car-object.md)):

- `CustomizeRims` → the wheel slots (four, usually driven together).
- `CustomizeSpoiler` → the spoiler slot.
- `CustomizePaint` → the paint slot (`CustomizePaintDatum` = the specific colour value written).
- `CustomizePerformance` → the performance slots (engine, brakes, …), one screen fanning into many
  ([C68.3](03-part-catalog.md)).

So a category is *the UI over one region of the slot map*: it shows the parts that can fill those slots and lets you
choose. Choosing writes the choice into the `PlayerCar`'s slot ([C68.4](04-buying.md)) — pending payment. This is
why the shop feels uniform across performance and visual despite their different *effects*: every category is the
same interaction (list the slot's options, pick one, add to cart), only the *consumer* of the result differs.

> 🟡 *Reasoned:* the mapping of each `Customize*` category to a car slot/slot-family follows from the screen names
> and the slot model ([C68.1](01-car-object.md)); the exact slot table per `CarType` is vault data. The screen names
> and the performance/visual split are verified.

## RE implications

- **A tree of `Customize*` screens** — `CustomizeMain` root → category screens → `CustomizePartOption` pickers.
- **Performance vs visual** — one deep performance category + several visual siblings — the top level *is* the
  two-customization split ([C56.1](../C56-Customization/01-two-customizations.md)).
- **A category = a slot family** — the UI over one region of the car's slot map; choosing writes the slot.
- **`CustomizeHUDColor`** customizes the UI ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)), grouped visually.

---

### Key takeaways

- The shop is a **tree of `Customize*` screens** — a `CustomizeMain` root branching to `CustomizePerformance` and
  the visual categories (`CustomizePaint`, `CustomizeRims`, `CustomizeSpoiler`, `CustomizeDecals`,
  `CustomizeNumbers`, `CustomizeHUDColor`), with `CustomizeParts`/`CustomizePartOption` as the per-slot pickers.
- The categories split into **performance** (one deep category → the part families,
  [Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)) and **visual** (the cosmetic
  siblings, [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)) — the shop's top level *is* the
  two-customization split.
- **Each category is the UI over a slot family** — same interaction everywhere (list options, pick, add to cart);
  only the *consumer* of the result (sim vs renderer) differs.
- **`CustomizeHUDColor`** is the outlier — it customizes the on-screen HUD tint
  ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)), not the car, but sits with the visual categories.
- All the `Customize*` screen names are **verified strings** in `speed.exe`.

**Continue:** [C68.3 — Parts as catalog entries](03-part-catalog.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md)
