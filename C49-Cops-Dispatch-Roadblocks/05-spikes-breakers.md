# C49.5 — Spikes & Pursuit Breakers

> **The one-sentence version:** two tools tip a pursuit — `SpikeStrip`/`SpikeBelt`, deployed across the road to
> puncture your tyres (`ETirePunctured`), the cops' weapon against you; and pursuit breakers, droppable
> environment that `ApplyBreakerZones` (`0x42E930`) turns into timed wreck-spheres disabling cops caught inside,
> your weapon against them.

[← C49.4 — Roadblocks](04-roadblocks.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md) ·
[Next: C49.6 — Reading the fleet in RE →](06-reading-fleet.md)

---

## Spike strips: the cops' tool

**Spike strips** (`SpikeStrip`/`SpikeBelt`, verified strings) are the cops' weapon against your tyres. Deployed
across the road (often with a roadblock, [C49.4](04-roadblocks.md)), a spike strip that you drive over **punctures
your tyres** — setting the `ETirePunctured` state ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md))
that drastically cuts grip. The effect is exactly what a real spike strip does: a crippled car, hard to control,
easy to catch.

- **Deployment** — a cop lays a spike strip on the road ahead of you (an "avoidable" registered with
  `AvoidableManager`, [Chapter 47](../C47-AI-Driver-Vehicle/04-navigation-systems.md)).
- **The hit** — driving over it punctures the tyres on that side (`ETirePunctured`,
  [Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) — grip loss, pulling, reduced top speed.
- **The counter** — swerve around it (it's a narrow strip), which is why it's usually paired with a roadblock
  (forcing you toward it) — a coordinated trap.

So spikes are the cops' way to *degrade your car* directly ([Chapter 45](../C45-Damage-Deformation/04-scaling-performance.md))
— not by ramming, but by wrecking your tyres. Late in a high-Heat pursuit, a spike hit can be what ends your run,
because a punctured car can't outrun the fleet.

> ✅ *Verified:* `SpikeStrip`, `SpikeBelt`, and `spikes` are present as strings in `speed.exe`; the tyre-damage
> states `ETirePunctured`/`ETireBlown` ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) are the
> spike outcome. Spikes register as "avoidables" for the AI to steer around.

## Pursuit breakers: your tool

The **pursuit breaker** is your counter-weapon: a droppable piece of environment (a gas station canopy, a water
tower, a giant sign) that you trigger, dropping it onto pursuing cops. Its mechanism is verified:
**`ApplyBreakerZones`** (`0x42E930`), one of `AICopManager`'s per-frame updates ([C49.1](01-fleet-manager.md)),
creates **timed wreck-spheres**:

- **Trigger** — driving through the breaker's trigger drops the object.
- **The zone** — a wreck-sphere (a radius) is created at the drop point, held in a list on the manager, and applied
  to **any cop inside its radius** — disabling them.
- **Timed** — the zone expires after a short time, so it disables the cops caught in the moment, then clears.

So a pursuit breaker is a *momentary area-disable* — trigger it as cops are behind you, and the ones in the
wreck-sphere are knocked out of the chase ([Chapter 45](../C45-Damage-Deformation/05-cop-damage.md),
`DamageCopCar`). This is the marquee "shake the cops" move — a satisfying, cinematic way to thin the pack, and the
counterpart to the cops' spikes.

> ✅ *Verified:* `ApplyBreakerZones` (`0x42E930`) is a verified `AICopManager` sub-update — it applies timed
> wreck-spheres (a list of radius zones) to cops caught inside, disabling them. This is the pursuit-breaker
> mechanism.

## The symmetry: spikes vs. breakers

Spikes and breakers make the pursuit a **two-way battle** of tools ([Chapter 45](../C45-Damage-Deformation/05-cop-damage.md)):

| | Cops' tool | Your tool |
|---|---|---|
| **Weapon** | spike strip (`SpikeStrip`) | pursuit breaker (`ApplyBreakerZones`) |
| **Effect** | punctures *your* tyres (`ETirePunctured`) | disables *their* cars (wreck-sphere) |
| **Placement** | on the road ahead (with roadblocks) | fixed environment objects |
| **Counter** | swerve around | trigger at the right moment |

This symmetry is deliberate: the cops have a way to cripple you (spikes), and you have a way to cripple them
(breakers). Neither side is helpless. The pursuit is a *contest* — they deploy spikes and roadblocks
([C49.4](04-roadblocks.md)); you deploy breakers and outrun them ([Chapter 48](../C48-Pursuit-Heat/04-bust-evade.md)).
This back-and-forth is what makes Most Wanted's pursuits feel like a *fight*, not just a chase — the defining
quality of the game, built from these verified mechanics.

## Why environmental tools

Making both tools *environmental* (spikes on the road, breakers as world objects) rather than abstract abilities is
a strong design ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)):

- **They're *placed* in the world.** Pursuit breakers are specific, learnable spots on the map — you *know* where
  the water tower is and lead cops to it. This rewards map knowledge.
- **They're *physical*.** A spike strip is a real object you can see and swerve; a breaker is a real object that
  physically falls ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) on cops. The tools obey the
  same physics as everything else.
- **They tie to the systems already built** — spikes to tyre damage
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)), breakers to cop damage
  ([Chapter 45](../C45-Damage-Deformation/05-cop-damage.md)) and the avoidance system
  ([Chapter 47](../C47-AI-Driver-Vehicle/04-navigation-systems.md)). No special-case code — they compose existing
  mechanics.

So spikes and breakers are the pursuit's tactical layer, built by *composing* the physics, damage, and world
systems rather than bolting on new ones — the same economy as the rest of the engine. They turn the map itself into
part of the chase.

## RE implications

- **Spikes** (`SpikeStrip`/`SpikeBelt`) are the cops' tool — puncture your tyres (`ETirePunctured`); counter by
  swerving.
- **Pursuit breakers** are your tool — `ApplyBreakerZones` (`0x42E930`) creates timed wreck-spheres disabling cops
  inside.
- **Symmetry** — cops cripple you (spikes), you cripple them (breakers) — a two-way battle.
- **Environmental** — both are physical world objects, composing the tyre/damage/avoidance systems, not new code.

---

### Key takeaways

- **Spike strips** (`SpikeStrip`/`SpikeBelt`) are the **cops' tool** — driving over one punctures your tyres
  (`ETirePunctured`), crippling grip; usually paired with roadblocks to force you onto them.
- **Pursuit breakers** are **your tool** — verified via `ApplyBreakerZones` (`0x42E930`), which creates **timed
  wreck-spheres** that disable any cop caught inside.
- The two make the pursuit a **two-way battle** — the cops can cripple you (spikes), you can cripple them
  (breakers); neither side is helpless.
- Both are **environmental** — physical, placed world objects that reward map knowledge and obey the same physics.
- They **compose existing systems** (tyre damage, cop damage, avoidance) rather than adding new code — the engine's
  economy applied to the pursuit's tactical layer.

**Continue:** [C49.6 — Reading the fleet in RE](06-reading-fleet.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md)
