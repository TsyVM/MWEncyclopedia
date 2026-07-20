# C71.5 — The Complete Car

> **The one-sentence version:** a complete car is the sum of every system in the book — a `CarType` model
> (geometry, textures, vault) instanced as a `PlayerCar`, configured by performance and visual builds, simulated by
> the physics and drawn by the renderer, saved as a small delta and moddable at the file level — the whole engine
> seen through one object.

[← C71.4 — Modding a car's files](04-modding-files.md) · [Chapter 71 hub](C71-Cars-End-To-End.md) ·
[Book index →](../README.md)

---

## The car as a cross-section of the engine

A single car touches nearly every system the book decodes — it's a *cross-section* of the whole engine:

- **Identifiers** ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)) — the car, its parts,
  and its assets are all hashed names.
- **Geometry** ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) — `GEOMETRY.BIN`'s `BASE`/`KIT` solids.
- **Textures** ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) — `TEXTURES.BIN`/`VINYLS.BIN`/`PREVINYL.BIN`.
- **Materials** ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) — paint and finish.
- **The vault** ([Chapters 11–13](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) — tuning values.
- **The object model** ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) — `PlayerCar` as a
  runtime object.
- **The sim** ([Chapters 39–42](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) — the physics reading the vault.
- **The renderer** ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) — drawing the geometry/textures/paint.
- **The save** ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)) — the build as a delta.
- **The front-end** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) — the `Customize*` shop screens.

To understand a car *completely* is to have understood all of these — which is why the cars cluster
([Chapters 68–71](C71-Cars-End-To-End.md)) sits late in the book: it *depends on* the formats, the vault, the sim,
and the renderer decoded earlier. The car is where they meet.

## The complete lifecycle

Following one car from nothing to a built, driven, saved, moddable machine traces the whole arc:

```
1. CarType exists          — shared model data (geometry, textures, base vault)      [Ch.5/8/13]
2. player acquires it      — a PlayerCar instance, saved as a selection              [C68.1, Ch.31]
3. performance build       — buy parts, climb tiers, bars rise                       [C71.2, Ch.68/69]
4. visual build            — kit, wheels, paint, livery                              [C71.3, Ch.70]
5. drive                   — the sim reads the vault, the renderer draws the build   [C71.1, Ch.42/51]
6. save                    — the build persists as a small per-save delta            [Ch.31]
7. mod (optional)          — edit the BINs/vault directly, size-neutral, verified    [C71.4, Ch.75]
```

Each step is a chapter (or several) of the book, and the car is the thread through them. This lifecycle is the
practical answer to "how does a car work in Most Wanted" — not one system, but the *composition* of many, each doing
its one job over the shared car data.

## Why the two-domain design is the key

If there's a single idea to carry away from the cars cluster, it's **one object, two data domains**
([C71.1](01-anatomy.md)): performance (vault → sim) and visual (file set → renderer), independent and never crossing.
This design is what makes MW's cars *work* the way they do:

- **Independence** — you can build a sleeper (slow-looking, fast) or a show car (fast-looking, slow) because the two
  domains don't constrain each other.
- **Honesty** — the garage bars can't lie about the drive ([C69.4](../C69-Performance-Upgrades-Tuning/04-upgrade-to-behaviour.md))
  because both read the *same* vault; the visual side is a separate, parallel truth.
- **Small saves** — the build is *selections* into shared model data
  ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)), so a garage of cars costs little to store.
- **Clean modding** — the domains map to distinct files ([C71.4](04-modding-files.md)), so a mod is a *targeted*
  edit to one of them.

The whole cars cluster is, in a sense, an extended argument that this two-domain object model is the elegant heart of
MW's customization — everything else (the shop, the catalog, the bars, the file set) is machinery serving it.

## Reading a car, end to end

To read *any* car in RE, the complete method is:

1. **Identify the `CarType`** — the model and its shared data ([C71.1](01-anatomy.md)).
2. **Read the file set** — `GEOMETRY.BIN` + the `TPK`s ([C70.5](../C70-Visual-Customisation/05-reading-visual.md)).
3. **Read the vault entries** — the car's tuning ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
4. **Trace the two consumers** — sim (vault) and renderer (file set) ([C71.1](01-anatomy.md)).
5. **Find the build** — the save's selections ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)).

Do this and you've read the car whole — model, build, behaviour, and appearance. The cars cluster gives you each
piece ([Chapters 68–70](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)); this chapter is the
map that assembles them.

## RE implications

- **A car is a cross-section of the engine** — identifiers, geometry, textures, vault, object model, sim, renderer,
  save, front-end all meet in one car.
- **The complete lifecycle** — `CarType` → `PlayerCar` → performance build → visual build → drive → save → mod.
- **Two-domain design is the key** — performance (vault→sim) and visual (files→renderer), independent — enabling
  sleepers, honest bars, small saves, clean modding.
- **Reading a car end to end** — `CarType` → file set → vault → two consumers → build.

---

### Key takeaways

- A complete car is the **sum of nearly every system** in the book — a **cross-section of the engine** — which is why
  the cars cluster sits late: it depends on the formats, vault, sim, and renderer decoded earlier.
- The **lifecycle** runs `CarType` (shared model) → `PlayerCar` (instance/save) → **performance build**
  ([C71.2](02-performance-build.md)) → **visual build** ([C71.3](03-visual-build.md)) → **drive** (sim + renderer) →
  **save** (delta) → **mod** ([C71.4](04-modding-files.md)) — each step a chapter, the car the thread.
- The **key design** is **one object, two data domains** ([C71.1](01-anatomy.md)) — performance (vault→sim) and
  visual (files→renderer), independent — which gives **sleepers**, **honest garage bars**, **small saves**, and
  **clean modding**.
- To **read any car**: identify the `CarType`, read the file set + vault, trace the two consumers, find the build —
  model, build, behaviour, and appearance, whole.
- This chapter is the **synthesis** — the map that assembles [Chapters 68–70](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)
  into one complete, moddable car.

**This completes the cars cluster (Chapters 68–71).** See the [book index](../README.md) for the full chapter map.

**Sources:** synthesis of the verified cluster — [Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)
(object: `PlayerCar`/`CarType`/`PART_*`), [Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)
(`PERF_*`, the `TOPSPEED`/`ACCELERATION`/`HANDLING` bars), [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)
(`GEOMETRY.BIN` `BASE`/`KIT`, `VINYLS.BIN`/`PREVINYL.BIN`, paint targets). Underlying formats:
[Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md) (`TPK`), [Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)
(solids), [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md) (vault). Sim & save:
[Chapters 39–42](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md), [Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md).
