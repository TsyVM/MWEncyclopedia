# C14.3 — Surface Records

> **The one-sentence version:** the ground is data — `carsurface` and `terraindriving` define, per ground
> type, the grip multipliers that make dirt slide and asphalt grip, plus the sounds and particle effects a
> surface throws — which is why leaving the road changes how the car handles, sounds, and looks.

[← C14.2 — The heat & bounty system](02-heat-bounty.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md) ·
[Next: C14.4 — Effects & destructibles →](04-effects-destructibles.md)

---

## Surfaces are collections

Two verified collections define how the world's surfaces behave:

- **`carsurface`** (`0xFDA45513`) — how a surface affects the *car*: grip/friction multipliers, drag, and the
  handling response of different ground types.
- **`terraindriving`** (`0x0AEE9EE6`) — the terrain-driving parameters for off-road/rough ground: how the car
  behaves when it leaves the asphalt.

Each is a vault collection inheriting from `default`
([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)); the surface types (asphalt, dirt, grass,
cobble, etc.) are entries whose fields set the physical and sensory response. The `SHD_*`/`TRNS_*` world
texture names ([C7.4](../C7-Materials-TexAnim/04-usage-names.md)) that tag ground meshes connect the visual
surface to its surface-type data.

## What a surface changes

A surface record parameterises three coupled things, which is why crossing onto dirt is a whole-sense change:

- **Grip / handling.** A friction/grip multiplier scales cornering and traction — high on asphalt, low on dirt
  and grass — so the same steering input slides on one surface and holds on another
  ([C13.4](../C13-Vault-CarTuning/04-value-to-sim.md)).
- **Sound.** The tire/road sound a surface produces (the `carsurface`-linked audio), so gravel sounds
  different from tarmac.
- **Effects.** The particle effects a surface throws — dust, leaves, sparks — named among the `fx*` instances
  ([C14.4](04-effects-destructibles.md)) and triggered as the car drives or slides on that surface.

So a surface type is a bundle of *feel*, *sound*, and *sight*, tuned together.

> ✅ *Verified:* `carsurface` (0xFDA45513) and `terraindriving` (0x0AEE9EE6) are present collections; surface
> effect instances (dust/leaves) appear among the `fx*` names.
> 🟡 *Reasoned:* the specific field that is "grip multiplier" versus "drag" is established by resolving field
> names and value plausibility; the collections and their role are verified.

## The surface pipeline

Surfaces tie together three subsystems you have already met:

```
world ground mesh (C8/C9)  → surface-type tag (texture usage name, C7.4)
        → surface record (carsurface / terraindriving)  → grip + sound + fx
```

The geometry says *where* a surface is; the usage name says *what type*; the surface record says *how it
behaves*. This is the world analogue of a car's material binding — a lookup from a tagged surface to its
data-driven response.

## Editing implications

- **Grip is the headline lever.** Raising a surface's grip multiplier makes that ground type easier to drive;
  lowering it makes it treacherous. This is the field most surface mods touch.
- **Keep the senses consistent.** If you make grass grippy, its sound and effects still say "grass" — coherent
  mods change grip *and* the sensory cues, or accept the mismatch.
- **Off-road lives in `terraindriving`.** Tune rough-ground behaviour there, not in `carsurface`.
- **Standard discipline.** Resolve, edit `Float` in place, verify ([C12.6](../C12-Reflection-Schema/06-writing-values.md)),
  and test by driving onto the surface.

---

### Key takeaways

- Surfaces are vault collections: `carsurface` (grip/handling per ground type) and `terraindriving` (off-road).
- A surface bundles grip, sound, and effects — crossing onto dirt changes all three.
- The pipeline is ground mesh → surface-type tag (usage name) → surface record → response.
- Grip multiplier is the main tuning lever; keep sound/effects coherent with it.
- Edit `Float` fields in place with the standard discipline and test on the surface.

**Continue:** [C14.4 — Effects & destructibles](04-effects-destructibles.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md)
