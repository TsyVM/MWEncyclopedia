# C68.5 — Reading Vehicles in RE

> **The one-sentence version:** navigate the customisable car by the object (`PlayerCar`/`CarType`/`CarSlot`/
> `CarPart`), the shop tree (`Customize*`), the catalog (`PART_*`), and the buy flow (`CustomizeShoppingCart`) —
> reading a car as an *instance configured by owned parts*, distinct from the vault data those parts select.

[← C68.4 — What "buying" does](04-buying.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md) ·
[Next: Chapter 69 — Performance Upgrades & Tuning Bars →](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)

---

## Anchors for vehicle RE

The customisable car is anchored on verified strings:

- **The object** — `PlayerCar`, `CarType`, `CarSlot`, `CarPart`, `CustomizePart` ([C68.1](01-car-object.md)).
- **The shop** — the `Customize*` screen tree (`CustomizeMain`, `CustomizePerformance`, `CustomizePaint`,
  `CustomizeRims`, `CustomizeSpoiler`, `CustomizeDecals`, `CustomizeNumbers`, `CustomizeHUDColor`,
  [C68.2](02-shop-categories.md)).
- **The catalog** — the `PART_<FAMILY>_*` family (nine families, [C68.3](03-part-catalog.md)).
- **The purchase** — `CustomizeShoppingCart`, `FEShoppingCartItem`, `ShoppingCart_BACKROOM` ([C68.4](04-buying.md)).

From these, the whole car-as-customisable-object is navigable: what a car is, how it's customised, what the parts
are, and what buying does.

## The RE workflow

Reading the customisable car:

1. **Find the object** — `PlayerCar`/`CarType` ([C68.1](01-car-object.md)); the instance and its model.
2. **Walk the shop tree** — `CustomizeMain` → category screens ([C68.2](02-shop-categories.md)).
3. **Read the catalog** — `PART_*` families ([C68.3](03-part-catalog.md)); the parts and their effects.
4. **Trace the buy flow** — `CustomizeShoppingCart` ([C68.4](04-buying.md)); owned + installed.

The output is the full customisation picture: a `PlayerCar` is a `CarType` plus a set of owned, installed parts,
chosen through the shop tree and committed through the cart.

## Object vs data

The key distinction this chapter draws is *object/flow* vs *data*:

- **This chapter** — the **object and flow**: what a car *is* (`PlayerCar`), how the shop is *organised*
  (`Customize*`), and what buying *does* (`CustomizeShoppingCart`).
- **[Chapter 56](../C56-Customization/C56-Customization.md) / [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)** —
  the **vault data** the parts select: the actual tuning numbers the sim reads.
- **[Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)** — the **visual assets** the visual parts
  select: the meshes, textures, and decal masks.

So reading a car completely means reading three layers: the *object* (this chapter — the slots and the buy flow),
the *tuning data* ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md) — what performance parts change), and
the *visual data* ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md) — what visual parts
change). This chapter is the *spine* they hang on: the object holds the slots, the parts fill them, and the buy flow
commits them — and the other chapters decode what each filled slot *means* to the sim and the renderer.

## Where the cars cluster goes next

This chapter opened the cars cluster with the *object*. The rest builds on it:

- **[Chapter 69 — Performance Upgrades & Tuning Bars](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)** —
  the *mechanics* of the performance families ([C68.3](03-part-catalog.md)): how each upgrade moves the
  `TOPSPEED`/`ACCEL`/`HANDLING` bars and the sim underneath them.
- **[Chapter 70 — Visual Customisation](../C70-Visual-Customisation/C70-Visual-Customisation.md)** — the *assets* of
  the visual categories ([C68.2](02-shop-categories.md)): body kits, wheels, paint, and the per-car decal masks.
- **[Chapter 71 — Cars, End to End](../C71-Cars-End-To-End/C71-Cars-End-To-End.md)** — the *walkthrough* that ties
  the object (this chapter), the tuning ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)),
  and the visuals ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)) into one modding pass.

## RE implications

- **Anchor on** the object strings, the `Customize*` tree, the `PART_*` catalog, and the cart.
- **The RE workflow** — object → shop tree → catalog → buy flow.
- **Object vs data** — this chapter (the object/flow) vs the vault
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and visual assets
  ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)).
- **The cluster** — object (68) → upgrades (69) → visuals (70) → end-to-end (71).

---

### Key takeaways

- The customisable car is anchored on **`PlayerCar`/`CarType`/`CarSlot`/`CarPart`** (object), the **`Customize*`**
  screen tree (shop), the **`PART_*`** families (catalog), and **`CustomizeShoppingCart`** (buy flow).
- The RE workflow: **find the object → walk the shop tree → read the catalog → trace the buy flow** — yielding a
  `PlayerCar` as a `CarType` plus owned, installed parts.
- **Object vs data** — this chapter is the **spine** (slots + buy flow); the **tuning data** is
  [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md), the **visual assets** are
  [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md) — three layers, read together for the whole
  car.
- The cars cluster continues: **upgrades & bars** ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)),
  **visual customisation** ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)), **end-to-end**
  ([Chapter 71](../C71-Cars-End-To-End/C71-Cars-End-To-End.md)).

**Next:** [Chapter 69 — Performance Upgrades & Tuning Bars](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md).

**Sources:** `speed.exe` (verified strings: `PlayerCar`, `CarType`, `CarSlot`, `CarPart`, `CustomizePart`; the
`Customize*` screen family — `CustomizeMain`/`MainOption`/`Sub`/`Category`/`Parts`/`PartOption`/`GenericTop`/
`Performance`/`Paint`/`PaintDatum`/`Rims`/`Spoiler`/`Decals`/`Numbers`/`HUDColor`; the `PART_<FAMILY>_*` catalog —
`EN`/`SU`/`EC`/`BR`/`WT`/`TR`/`TU`/`TI`/`NO`; `CustomizeShoppingCart`, `FEShoppingCartItem`, `ShoppingCart_BACKROOM`,
`ShoppingCart_QR`). Vault tuning data: [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md) /
[Chapter 56](../C56-Customization/C56-Customization.md). Object model: [Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md).
