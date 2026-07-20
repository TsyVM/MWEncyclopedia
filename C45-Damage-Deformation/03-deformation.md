# C45.3 — Deformation & Breakables

> **The one-sentence version:** damage shows two ways — coarse zones drive progressive **mesh crumple** (panels
> sag as a zone accumulates energy), and named parts fire **discrete breakage** (glass shatters, bumpers detach,
> lights go dark) when their zone crosses a threshold.

[← C45.2 — Damage zones](02-damage-zones.md) · [Chapter 45 hub](C45-Damage-Deformation.md) ·
[Next: C45.4 — Scaling & performance loss →](04-scaling-performance.md)

---

## Two kinds of visible damage

The zone systems ([C45.2](02-damage-zones.md)) produce two visibly different kinds of damage, matching their two
natures:

- **Crumple (continuous).** As a coarse `DAMAGE0_*` zone accumulates impact energy, the body mesh in that region
  **deforms progressively** — panels dent and fold, deeper with more damage. This is smooth and analog: a light
  tap barely marks the panel; a heavy hit caves it in.
- **Breakage (discrete).** As a zone's damage crosses **thresholds**, named parts ([C45.2](02-damage-zones.md))
  switch from intact to broken — glass shatters, a bumper detaches, a headlight goes dark. This is a snap: the
  part is whole, then it isn't.

Both play out from the same accumulating zone damage — the crumple is the continuous readout, the breakages are the
discrete milestones along the way. A crash is both at once: the front-left caves in *and* the headlight pops.

## Mesh crumple

The coarse-zone crumple is **mesh deformation** — the car's geometry
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) is pushed around to show the dent:

- **Progressive.** The deformation scales with the zone's accumulated damage — the more energy that zone has
  absorbed, the more its panels are displaced.
- **Directional.** Because each zone is a region ([C45.2](02-damage-zones.md)), the crumple appears where you were
  hit — a front-left impact folds the front-left, not the whole car.
- **Bounded.** There's a maximum deformation (a fully-crumpled zone) — damage saturates rather than folding the
  car infinitely.

This is what makes a battered Most Wanted car *look* battered — the panels genuinely deform, so a car that's been
through a long pursuit ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) wears its history. `DamageVehicle`'s
mesh-manipulation methods ([C45.1](01-damage-family.md)) are a chunk of its 127 — deforming geometry is real work.

> 🟡 *Reasoned:* the progressive mesh-deformation model (zone damage → panel vertex displacement, bounded) is the
> standard car-crumple technique, consistent with `DamageVehicle`'s large method count and the verified coarse
> zones; the exact deformation math is deeper RE. The zones and breakable parts are verified strings.

## Breakables and their states

The named parts ([C45.2](02-damage-zones.md)) are the **discrete** damage — each a small state machine (intact →
broken/detached), fired by its zone's damage:

- **Glass** — `WINDSHIELD`, `DAMAGE_FRONT_WINDOW`, `LEFT_HEADLIGHT_GLASS` — shatters (swaps to a cracked/absent
  mesh) when hit hard enough. The verified `BREAK_HEADLIGHT_LEFT`/`_RIGHT` are the headlight-break events.
- **Detachable panels** — bumpers (`DAMAGE_FRONT_BUMPER`), hood (`DAMAGE_HOOD`), doors (`DAMAGE_LEFT_DOOR`) — can
  deform and, past a threshold, **fall off** (become a separate `SimpleRigidBody`
  [C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md), or simply vanish).
- **Lights** — headlights and brakelights break and go dark, affecting the car's night appearance
  ([Chapter 25](../C27-FrontEnd-Shell-UI/04-hud.md) / lighting).

Each part's break is a threshold event on the accumulating zone damage — which is why parts break in a believable
order (a hard front hit breaks the front bumper and headlights before the hood detaches). The breakables are the
*drama* of damage — the shatter and the shed that read instantly as "that hurt."

## Deformation is presentation (mostly)

Deformation and breakage are largely **presentation** — the *visual* consequence of damage, in the DRAW-adjacent
side of the DAMAGE mechanic ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)):

- **Crumple and breakage show the damage** — they're the visible readout of the zone accumulators.
- **They mostly don't change physics** — a dented panel or a lost bumper doesn't itself alter handling; that's the
  *performance* side ([C45.4](04-scaling-performance.md)), a separate consequence of the same damage.
- **Exception: functional parts.** Some breakages have gameplay meaning — a broken headlight dims your night view,
  a lost cop light bar ([C45.5](05-cop-damage.md)) signals a disabled cruiser.

So deformation is the *look* of damage, running alongside the *performance* effect ([C45.4](04-scaling-performance.md))
— the same accumulating zone damage drives both the visible crumple/breakage and the mechanical degradation, but
they're distinct reads (like the surface's three reads, [C44.5](../C44-Surfaces-Grip/05-three-reads.md)). A car can
look wrecked and drive fine, or (with enough damage) both.

## RE implications

- **Two kinds of visible damage** — continuous **mesh crumple** (coarse zones) and discrete **part breakage**
  (named parts).
- **Crumple** is progressive, directional, bounded mesh deformation — the car wears its damage history.
- **Breakables** are per-part intact→broken state machines fired by zone-damage thresholds (glass, panels,
  lights).
- **Deformation is mostly presentation** — the *look* of damage, separate from the *performance* effect
  ([C45.4](04-scaling-performance.md)); a few parts (headlights, cop lights) are functional.

---

### Key takeaways

- Damage shows **two ways**: continuous **mesh crumple** (coarse `DAMAGE0_*` zones deform panels progressively) and
  discrete **breakage** (named parts snap from intact to broken).
- **Crumple** is progressive, directional, and bounded — a battered car genuinely looks battered, wearing its
  pursuit history.
- **Breakables** (glass `WINDSHIELD`/`*_HEADLIGHT_GLASS`, panels, lights) are per-part state machines fired at
  zone-damage **thresholds** — parts break in a believable order.
- Deformation is **mostly presentation** — the visual readout of the zone accumulators — running alongside the
  separate **performance** effect ([C45.4](04-scaling-performance.md)).
- A few breakages are **functional** (dark headlights, a knocked-off cop light bar,
  [C45.5](05-cop-damage.md)) — damage that means something in gameplay.

**Continue:** [C45.4 — Scaling & performance loss](04-scaling-performance.md) · [Chapter 45 hub](C45-Damage-Deformation.md)
