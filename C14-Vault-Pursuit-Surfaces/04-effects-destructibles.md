# C14.4 — Effects & Destructibles

> **The one-sentence version:** the reactive world is vault-driven — collision-world event collections and
> smackable/destructible records decide what happens when you hit something, and the hundreds of `fx*`
> instances name the visual effects those events fire.

[← C14.3 — Surface records](03-surfaces.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md) ·
[Next: C14.5 — The gameplay & FE_ATTRIB vaults →](05-other-vaults.md)

---

## Collision-world events

When you clip a sign, plough through a fence, or ram traffic, the game consults **collision-world event**
collections to decide the reaction. The vault's string table holds a family of these:
`collisionworld.interrupts`, `collworld_spin`, `collworld_flip`, `collworld_air`, `collworld_civi`
([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)) — "anytime events" that fire on collision to spin, flip,
launch, or otherwise perturb the car and the struck object. They are the rules of impact: what a hit *does*.

Each is a collection ([C12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) whose fields tune the event's
force and outcome — how much a smackable object launches, how a spin plays out. Editing them changes the
feel of crashing through the world.

## Smackables and destructibles

"Smackable" props — the breakable scenery MW is full of — are governed by records that define their mass,
break behaviour, and the effect they spawn when hit (`busstopsmack`, `carhitsmackable`, and the many
scenery-collision names). The world geometry ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) places
these objects; the vault says how they *react*. So a destructible is a partnership: geometry for the shape,
attributes for the physics of breaking.

## The fx instances

The most numerous vault entries are **effect instances** — the `fx*` names that dominate the string table:
`fxcar_impactdebris`, `fxcar_coplightblue`, `fxenv_leaffall_hvy`, `fxenv_smokestack`, `fxgame_flare_red`,
`fxnis_extradust1`, `fxtd_dr_asphalt_leaves`. The prefixes namespace them by domain:

| Prefix | Domain | Example |
|---|---|---|
| `fxcar_` | car effects | `fxcar_impactdebris` (crash debris) |
| `fxenv_` | environment | `fxenv_leaffall_hvy` |
| `fxgame_` | gameplay | `fxgame_flare_red`, `fxgame_icon_pursuit` |
| `fxnis_` | cutscene (NIS) | `fxnis_extradust1` |
| `fxtd_` | terrain-driving | `fxtd_dr_asphalt_leaves` |

An event (a collision, a surface contact, a pursuit escalation) references an `fx*` instance by name/hash; the
instance describes the particle effect to play. This is the same reference indirection as textures
([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)) — the event names *what* effect, the instance defines
*how* it looks.

> ✅ *Verified:* collision-event collections (`collisionworld.*`, `collworld_*`) and a large `fx*` instance
> population are present in the vault, namespaced by the prefixes above.
> 🟡 *Reasoned:* the precise field-level tuning of each event/effect is established by resolving field names;
> the collections, instances, and their roles are verified.

## How the pieces connect

The reactive world is a chain of vault lookups:

```
you hit a prop → collision-world event (what happens) → smackable record (how it breaks)
              → fx* instance (what it looks like)  +  surface/sound (C14.3)
```

Every arrow is a reference resolved through the vault, which is why the whole reactive layer is *tunable
data*: change the event to alter the physics of impact, the smackable to change what breaks, the `fx*`
instance to change the visual.

## Editing implications

- **Event fields tune impact feel** — spin/flip/launch strength lives in the `collworld_*`/`interrupts`
  collections.
- **Smackable records tune what breaks and how** — mass and break thresholds.
- **`fx*` instances tune the look** — edit the instance to change a debris burst or light effect; the event
  still references it by name.
- **Follow the references.** A missing effect is a dangling reference (the event names an `fx*` that isn't
  there), not a broken renderer — check the instance exists.

---

### Key takeaways

- Collision-world event collections (`collisionworld.interrupts`, `collworld_spin/flip/air/civi`) decide what
  a hit does.
- Smackable/destructible records govern how props break; geometry gives the shape, the vault the reaction.
- The numerous `fx*` instances (namespaced `fxcar_`/`fxenv_`/`fxgame_`/`fxnis_`/`fxtd_`) name the visual
  effects events fire.
- The reactive world is a chain of vault references: event → smackable → fx (+ surface/sound).
- Tune events for feel, smackables for breakage, `fx*` for looks; a missing effect is a dangling reference.

**Continue:** [C14.5 — The gameplay & FE_ATTRIB vaults](05-other-vaults.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md)
