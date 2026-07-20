# C52.3 — The FX Catalogue

> **The one-sentence version:** which effect plays is a catalogue lookup — the `fxtd_*` terrain-drive effects keyed
> by surface × tyre mode (`fxtd_sl_asphalt`, `fxtd_dr_sand`) and the `fxcar_*` impact effects keyed by collision
> type — the grid that maps every driving and crash situation to its particle effect.

[← C52.2 — Emitters & particles](02-emitters-particles.md) · [Chapter 52 hub](C52-Effects-Particles.md) ·
[Next: C52.4 — Per-entity effects →](04-entity-effects.md)

---

## Naming the effects

The particle effects ([C52.2](02-emitters-particles.md)) are named in a **structured catalogue** — the effect
names encode *what* effect and *when* it plays. Two major families:

- **`fxtd_*`** (fx **t**errain-**d**rive) — the effects of *driving on a surface*
  ([Chapter 44](../C44-Surfaces-Grip/04-tire-effects.md)): `fxtd_<mode>_<surface>`.
- **`fxcar_*`** (fx **car**) — the effects of a *car event* (impacts, [Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)):
  sparks, impact bursts.

The naming *is* the lookup key — a situation (surface + tyre mode, or collision type) maps to an effect name, which
names the emitter group ([C52.2](02-emitters-particles.md)) to play. So the catalogue is a *dictionary* from
game-situation to effect, encoded in the names.

> ✅ *Verified:* the `fxtd_*`/`fxcar_*` effect families and the surface×mode grid are catalogued in the effect
> bank / ErtS data (referenced by `TireEffectRecord`, [Chapter 44](../C44-Surfaces-Grip/04-tire-effects.md), which
> is a vault key ×50). The naming convention is the lookup structure.

## The surface × mode grid

The `fxtd_*` family is a **two-dimensional grid** ([C44.4](../C44-Surfaces-Grip/04-tire-effects.md)) — *surface* ×
*tyre mode*:

| | driving (`dr`) | skid (`sk`) | slide (`sl`) | fly/hit |
|---|---|---|---|---|
| asphalt | `fxtd_dr_asphalt` | `fxtd_sk_asphalt` | `fxtd_sl_asphalt` | `fxtd_hit_asphalt` |
| sand | `fxtd_dr_sand` | `fxtd_sk_sand` | `fxtd_sl_sand` | … |
| grass | `fxtd_dr_grass` | … | … | … |
| … | … | … | … | … |

Every cell is a distinct effect: rolling on asphalt (`fxtd_dr_asphalt`, faint) vs. a drift on asphalt
(`fxtd_sl_asphalt`, white smoke) vs. rolling on sand (`fxtd_dr_sand`, dust). This grid is *why* `TireEffectRecord`
is referenced ×50 ([C44.4](../C44-Surfaces-Grip/04-tire-effects.md)) — it's one entry per cell, and the grid is
large (surfaces × modes). The catalogue *is* this grid, filled in: the effect bank exists to hold one emitter group
per surface×mode, so every combination of "what surface, what the tyre's doing" has its exact visual.

## The lookup at runtime

At runtime, the effect for a situation is chosen by *building the key and looking it up*
([C44.4](../C44-Surfaces-Grip/04-tire-effects.md)):

```
tyre on surface S, doing mode M:
   effect_name = "fxtd_" + mode_code(M) + "_" + surface_name(S)   // e.g. "fxtd_sl_asphalt"
   emitter_group = fx_bank.lookup(effect_name)                     // the particle effect
   spawn emitter_group at the wheel                                 // C52.2
```

So the tyre model ([C44.4](../C44-Surfaces-Grip/04-tire-effects.md)) produces the surface and mode, the name is
composed, the catalogue is looked up, and the resulting emitter group spawns particles
([C52.2](02-emitters-particles.md)) at the wheel. The same pattern handles `fxcar_*` for impacts — the collision
type ([C43.3](../C43-Collision-Contacts/03-classification.md)) composes the key. This composed-name lookup is a
clean, data-driven dispatch: add a surface, add its `fxtd_*` rows, and the effects appear with no code change.

> 🟡 *Reasoned:* the composed-name lookup mechanism is the natural reading of the `fxtd_<mode>_<surface>` naming and
> the `TireEffectRecord` keying ([Chapter 44](../C44-Surfaces-Grip/04-tire-effects.md)); the exact bank format and
> lookup implementation are per-file RE. The effect families and the surface×mode grid structure are established
> from the naming and the ×50 record count.

## Why a named catalogue

Structuring effects as a named, composable catalogue (rather than hard-coded effect assignments) is the same
data-driven economy as the rest of the engine ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)):

- **Complete coverage by construction.** Every surface×mode cell has an effect because the grid is filled in — no
  situation is left without a visual, and none is special-cased in code.
- **Extensible by data.** A new surface ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) needs only its
  `fxtd_*` rows added; the composed-name lookup finds them automatically.
- **Consistent with the other reads.** The effect catalogue keys on the *same* surface tag as the grip and sound
  ([C44.5](../C44-Surfaces-Grip/05-three-reads.md)) — so a surface's feel, sound, and *look* stay coherent
  (the three synchronized reads).

So the FX catalogue is the *visual* third of the surface system ([C44.5](../C44-Surfaces-Grip/05-three-reads.md)),
realised as a named grid of emitter groups. It's how every drift, skid, and dust plume gets *exactly* the right
particles — the catalogue mapping situation to effect, composed at runtime, spawned from the pools
([C52.2](02-emitters-particles.md)).

## RE implications

- **The FX catalogue** maps situations to effects — `fxtd_*` (surface×mode) and `fxcar_*` (impact type).
- **The `fxtd_*` grid** is surface × tyre mode — one emitter group per cell (why `TireEffectRecord` is ×50).
- **Composed-name lookup** — build `fxtd_<mode>_<surface>`, look it up, spawn the group — data-driven dispatch.
- **The visual third** of the surface system — keyed on the same tag as grip and sound (coherent reads).

---

### Key takeaways

- Which effect plays is a **catalogue lookup** — `fxtd_*` (terrain-drive, keyed by *surface × tyre mode*) and
  `fxcar_*` (car impacts, keyed by *collision type*).
- The `fxtd_*` family is a **two-dimensional grid** (surface × mode) — every cell a distinct effect
  (`fxtd_sl_asphalt` = drift smoke, `fxtd_dr_sand` = dust) — which is why `TireEffectRecord` is referenced **×50**.
- The runtime **composes the name** (`fxtd_<mode>_<surface>`), looks it up in the bank, and spawns the emitter
  group at the wheel — clean data-driven dispatch.
- The catalogue is the **visual third** of the surface system ([C44.5](../C44-Surfaces-Grip/05-three-reads.md)) —
  keyed on the same surface tag as grip and sound, keeping the three reads coherent.
- Adding a surface needs only its `fxtd_*` **rows** — the effects appear with no code change (data-driven, complete
  by construction).

**Continue:** [C52.4 — Per-entity effects](04-entity-effects.md) · [Chapter 52 hub](C52-Effects-Particles.md)
