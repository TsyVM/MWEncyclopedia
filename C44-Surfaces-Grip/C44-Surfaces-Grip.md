# Chapter 44 — Surfaces: Grip, Sound & Effects

> **Goal of this chapter:** decode driving *on* the world — the surface taxonomy (`asphalt`, `grass`, `concrete`,
> `sand`…), and the three continuous reads of the surface tag: **grip** (handling), **`RoadNoiseRecord`** (sound),
> and **`TireEffectRecord`** (smoke/debris) — all verified vault records.

A collision is a moment ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)); a surface is a
*condition* the tyres sit in every frame. This chapter decodes the **continuous** side of touching the world: the
surface each wheel is on, and how that one tag fans out to three systems at once — how much grip the tyre has, what
the tyre sounds like, and what it kicks up. It's why every surface in Most Wanted *drives, sounds, and looks* like
itself — and it mirrors the collision fan-out ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)),
discrete there, continuous here.

> **Verified against the vault.** The surface tags are reflection-hash keys in `GLOBAL/attributes.bin`: `concrete`
> (`0x6CA26F9B`, ×23), `grass` (`0x772FB736`, ×7), `asphalt` (`0x19DB2F1E`, ×4), `sand` (`0x999ACD78`, ×4), `water`
> (`0x5A2E0437`, ×4), `dirt` (×2), `ice` (×2), `gravel`/`cobble`/`snow`/`mud` (×1 each). The two presentation
> records are heavily used: `rh("RoadNoiseRecord")=0xFFDB013B` appears **×48** and
> `rh("TireEffectRecord")=0x681D219C` appears **×50** — the tyre-sound and tyre-effect records, one entry per
> surface × mode. The tyre modes (driving/skid/slide/fly/hit) drive the `fxtd_*` effect selection.

---

## Deep-dive pages

- [C44.1 — The surface taxonomy](01-surface-taxonomy.md): the verified surface tags and their frequencies.
- [C44.2 — Grip: the functional read](02-grip.md): how the surface scales tyre grip — where the car goes.
- [C44.3 — RoadNoiseRecord: the sound](03-road-noise.md): the tyre roll/skid audio per surface (×48).
- [C44.4 — TireEffectRecord: the visual](04-tire-effects.md): the smoke/debris per surface × mode (×50).
- [C44.5 — The three synchronized reads](05-three-reads.md): one tag → grip + sound + visual, the continuous
  fan-out.
- [C44.6 — Reading surfaces in RE](06-reading-surfaces.md): navigating the surface system.

---

## 44.1 The surface taxonomy

Each wheel is always on *some* surface, identified by a **tag** ([C44.1](01-surface-taxonomy.md)) — a verified set
of vault keys: `concrete`, `asphalt`, `grass`, `sand`, `gravel`, `dirt`, `cobble`, `water`, `snow`, `mud`, `ice`.
Their frequencies tell a story: `concrete` (×23) and `grass` (×7) dominate — Rockport is a city, mostly paved with
grassy verges. The tag is set by the wheel's ground contact ([C43.1](../C43-Collision-Contacts/01-detection.md))
and read by three systems.

## 44.2 Grip

The **functional** read is grip ([C44.2](02-grip.md)): the tyre model
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) uses the surface to **scale
available grip** — asphalt holds, sand/grass/ice slide. This is the one read that changes *where the car goes*: on
grass you understeer and wash out; on ice you slither; on asphalt you grip. Grip is the surface's effect on the
physics ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)).

## 44.3 RoadNoiseRecord

The **audio** read is `RoadNoiseRecord` (×48, [C44.3](03-road-noise.md)): it selects the tyre roll/skid sample set
for the surface, so asphalt hums, gravel rattles, and grass goes soft. It's the reason you can tell what you're
driving on with your eyes closed. Forty-eight references — one per surface (× a few modes) — make it one of the
most-referenced audio records.

## 44.4 TireEffectRecord

The **visual** read is `TireEffectRecord` (×50, [C44.4](04-tire-effects.md)): it maps a (surface × tyre mode) to
the right `fxtd_*` terrain-driving effect — the smoke and debris. The **mode** is what the tyre is *doing*:
driving (`fxtd_dr_*`), skid (`fxtd_sk_*`), slide (`fxtd_sl_*`), fly/hit (`fxtd_fly_*`/`fxtd_hit_*`
[Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)). Cross the mode with the surface and you get the
exact effect: a drift on asphalt → `fxtd_sl_asphalt`; rolling on sand → `fxtd_dr_sand`.

## 44.5 Three synchronized reads

The chapter's thesis ([C44.5](05-three-reads.md)): **one surface tag, three synchronized reads.**

```
tyre on surface S, doing mode M
   ├──▶ grip            (tyre model)         →  hold vs. slide
   ├──▶ RoadNoiseRecord (audio)              →  roll/skid sound for S
   └──▶ TireEffectRecord → fxtd_<M>_<S>      →  smoke/debris for S × M
```

All three branch on the same tag, so a `sand` surface picks low grip, sandy tyre noise, and `fxtd_dr_sand`
together — the surface is coherent across feel, sound, and look. This mirrors the collision fan-out
([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)): tag → physics + sound + visuals.

---

### Key takeaways

- Each wheel is on a **surface tag** — verified vault keys `concrete` (×23), `grass` (×7), `asphalt`, `sand`,
  `gravel`, `dirt`, `cobble`, `water`, `snow`, `mud`, `ice`.
- The tag drives **three continuous reads**: **grip** (the functional one — where the car goes),
  **`RoadNoiseRecord`** (×48, sound), **`TireEffectRecord`** (×50, smoke/debris).
- `TireEffectRecord` keys on **surface × tyre mode** (driving/skid/slide/fly/hit) → the `fxtd_*` effects — a drift
  on asphalt and a roll on sand differ.
- **One tag, three synchronized reads** — a surface is coherent across feel, sound, and look, the *continuous*
  counterpart to the collision fan-out.
- The frequencies reflect the world: **concrete and grass dominate** a paved city with grassy verges.

**Next:** [Chapter 45 — Damage & Deformation](../C45-Damage-Deformation/C45-Damage-Deformation.md): the other
consequence of touching the world.
