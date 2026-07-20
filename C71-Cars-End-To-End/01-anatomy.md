# C71.1 — Anatomy of a Car

> **The one-sentence version:** a car is one object (`PlayerCar` of a `CarType`) holding two independent
> configurations — a *performance* config that selects vault values the sim reads, and a *visual* config that
> selects meshes, textures, and colours the renderer reads — so the whole car is the object plus its two data
> domains.

[← Chapter 71 hub](C71-Cars-End-To-End.md) · [Next: C71.2 — The performance build →](02-performance-build.md)

---

## One object, two domains

The single most important fact about MW's cars — established across the cluster and worth stating whole — is that a
car is **one object with two data domains**:

```
                         PlayerCar  (a CarType + your config)
                        /                                    \
        performance config                              visual config
        (installed PART_* parts)                        (active KIT, paint, vinyls)
                │                                             │
                ▼                                             ▼
        vault values (Ch.13)                          file set (Ch.5/8, C70)
                │                                             │
                ▼                                             ▼
        the SIM reads them (Ch.42)                    the RENDERER reads them (Ch.51)
```

The `PlayerCar` ([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)) is the *instance* of a `CarType` (the
model). It carries a **performance config** — which `PART_*` parts ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md))
are installed, selecting tuning values in the vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — and
a **visual config** — which body kit, wheels, paint, and vinyls ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md))
are active, selecting from the file set. The two are *independent*: a slow car can look fast, a fast car can look
stock.

## The two consumers

Each domain has exactly one consumer, and they never cross:

- **The sim** ([Chapters 39–42](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) reads the **vault** — torque,
  gearing, grip, brake force, mass ([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)) — to produce
  driving *behaviour*. It never reads a mesh or a colour.
- **The renderer** ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) reads the **file set** — the active
  `GEOMETRY.BIN` solids, the `TPK` textures, the paint parameters ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md))
  — to produce *appearance*. It never reads a tuning number.

This clean split is why the two customizations ([C56.1](../C56-Customization/01-two-customizations.md)) are truly
independent: they write to different data, read by different consumers. It's also why the garage bar can be a faithful
preview of the drive ([C69.4](../C69-Performance-Upgrades-Tuning/04-upgrade-to-behaviour.md)) — the bar summarises the
*same* vault the sim reads — while the visual side is a separate, parallel story.

## The car's data at rest

Where does a car's data *live*? Across three places, by domain:

- **The model** (shared, read-only) — the `CarType`'s `GEOMETRY.BIN` ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)),
  `TEXTURES.BIN`/`VINYLS.BIN`/`PREVINYL.BIN` ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), and base vault
  entries ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). Every player's M3 GTR shares these.
- **The build** (per-save) — *your* installed parts, active kit, paint, and vinyls, recorded in the save
  ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)) as *selections* into the shared model data.
- **The runtime object** (live) — the `PlayerCar` in memory ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)),
  assembled from the model + the build when the car is spawned.

So a saved car is *small* — a set of selections, not a copy of the meshes — because the heavy data (geometry,
textures) is shared read-only model data ([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)). This is the
type/instance split ([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)) at the storage level: the
`CarType` is the shared bulk, the `PlayerCar` build is the tiny per-save delta.

## RE implications

- **One object, two domains** — the `PlayerCar` holds a performance config (vault) and a visual config (file set),
  independent.
- **Two consumers** — the sim reads the vault, the renderer reads the file set; they never cross.
- **Data at rest** — shared model (read-only) + per-save build (selections) + live object (assembled) — the
  type/instance split at storage level.
- **A saved car is small** — selections into shared model data, not a copy.

---

### Key takeaways

- A car is **one object, two data domains** — the `PlayerCar` ([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md))
  holds a **performance config** (installed parts → vault, [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
  and a **visual config** (kit/paint/vinyls → file set, [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)),
  **independent** of each other.
- **Two consumers, never crossing** — the **sim** ([Chapters 39–42](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
  reads the vault (behaviour), the **renderer** ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) reads
  the file set (appearance).
- A car's data lives in **three places** — the shared read-only **model** (`CarType`), the per-save **build**
  (selections, [Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)), and the live **object** (assembled).
- A **saved car is small** — a set of selections into shared model data, not a copy of the meshes — the
  type/instance split at the storage level.

**Continue:** [C71.2 — The performance build](02-performance-build.md) · [Chapter 71 hub](C71-Cars-End-To-End.md)
