# C70.1 — Body Kits as Geometry

> **The one-sentence version:** a body kit is not a parameter but *alternate geometry* — the car's `GEOMETRY.BIN`
> stores a `BASE` body plus one or more `KIT` variants (each with its own body panels, damage stages, decal meshes,
> and front parts), and choosing a kit swaps which solid objects the car renders.

[← Chapter 70 hub](C70-Visual-Customisation.md) · [Next: C70.2 — Wheels, brakes & aero →](02-wheels-aero.md)

---

## A car's geometry is a set of alternates

Open a car's `GEOMETRY.BIN` ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) and you don't find *one* car
mesh — you find a **library of alternates**. The retail `CARS/BMWM3GTR/GEOMETRY.BIN` holds ~97 solid objects
organised into a `BASE` set and `KIT` variants:

```
BMWM3GTR_BASE_A .. BASE_D           — the base body panels
BMWM3GTR_KIT00_BODY_A .. BODY_E     — body-kit 00's panels
BMWM3GTR_KIT00_DAMAGE0_FRONTLEFT_*  — that kit's damage stages
BMWM3GTR_KIT00_DECAL_LEFT_DOOR_*    — that kit's decal meshes
BMWM3GTR_KIT00_DRIVER_A / _C        — the driver model
BMWM3GTR_KIT00_FRONT_BRAKE_*        — front brake/tyre/window parts
```

So the car is a *menu of body pieces*, and a "body kit" is a **named subset** of that menu — `KIT00` is one complete
alternate body, with all the panels, damage geometry, and trim that go with it. A car with multiple kits
(`KIT00`, `KIT01`, …) is carrying several complete bodies in one `GEOMETRY.BIN`, and choosing a kit selects which
set of solids to draw.

> ✅ *Verified:* retail `CARS/BMWM3GTR/GEOMETRY.BIN` is a solid-object bundle
> ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) whose object names group into a `BASE` family and
> `KIT00_*` families (`BODY`, `DAMAGE`, `DECAL`, `DRIVER`, `FRONT`); `speed.exe` names `BODYKIT` and `KIT` (×8).

## What a kit bundles

A `KIT` is not just body panels — it's *everything that changes with the body*:

- **`KIT00_BODY_*`** — the panels themselves (the visible shell, split into parts `A`–`E` for damage/LOD).
- **`KIT00_DAMAGE0_*`** — the damage geometry ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md))
  *for that kit* — front-left, front-right, rear, etc. Each kit has its own crumple meshes, because a wide-body kit
  deforms differently than the base.
- **`KIT00_DECAL_*`** — the decal-mount meshes (where vinyls/decals sit, [C70.4](04-vinyls-decals.md)) shaped to that
  kit's surfaces.
- **`KIT00_FRONT_*`** / **`KIT00_DRIVER_*`** — the wheels-area parts and driver, bundled so the whole car is
  consistent.

So a body kit is a *coherent visual package*: change the body and its damage, decals, and trim all change with it.
This is why body kits in MW feel like whole transformations rather than bolt-ons — selecting `KIT00` swaps a
*complete, self-consistent body* including how it breaks and where decals land.

## Geometry, not parameter

The key structural fact is that body kits are **geometry swaps, not vault values** — the opposite of performance
parts ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)):

- A **performance part** changes a *number* the sim reads ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
  — same mesh, different behaviour.
- A **body kit** changes the *mesh* the renderer draws — different appearance, same behaviour (kits are cosmetic; a
  wide body doesn't corner better).

So the two customizations ([C56.1](../C56-Customization/01-two-customizations.md)) reach *different data*:
performance edits the vault, visual edits which *solids* are active. The car object
([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)) holds both — a performance config *and* a visual
config (active kit) — and its two consumers read the two: the sim reads the vault, the renderer reads the active
geometry. Reading body kits correctly means seeing them as the *renderer's* half of the slot model: a visual slot
whose "part" is a mesh set in `GEOMETRY.BIN`.

## RE implications

- **A car's `GEOMETRY.BIN` is a library of alternates** — a `BASE` set plus `KIT` variants
  ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)).
- **A kit bundles everything** — body panels, damage geometry
  ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)), decal meshes, front parts, driver — a coherent
  package.
- **Geometry, not parameter** — body kits swap *meshes* (renderer), performance parts swap *numbers* (sim); the two
  customizations reach different data.
- **The renderer's half of the slot model** — a visual slot whose part is a mesh set.

---

### Key takeaways

- A car's `GEOMETRY.BIN` ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) is a **library of alternate
  bodies** — a `BASE` set plus `KIT` variants — so a **body kit is a named subset of solid objects**, not a
  parameter.
- A `KIT` bundles a **coherent visual package**: body panels, its own **damage geometry**
  ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)), decal-mount meshes, front parts, and driver —
  which is why a kit feels like a whole transformation.
- Body kits are **geometry swaps, not vault values** — the mirror of performance parts
  ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)): visual customization edits *which solids are
  active*, performance edits *numbers*.
- The car object ([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)) holds **both** configs; its two
  consumers read them — the **renderer** reads the active geometry, the **sim** reads the vault.
- Verified: the `BASE`/`KIT00_*` geometry grouping in retail car files and `speed.exe`'s `BODYKIT`/`KIT` strings.

**Continue:** [C70.2 — Wheels, brakes & aero](02-wheels-aero.md) · [Chapter 70 hub](C70-Visual-Customisation.md)
