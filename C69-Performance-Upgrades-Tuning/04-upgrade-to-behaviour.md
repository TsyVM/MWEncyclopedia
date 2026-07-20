# C69.4 — From Upgrade to Bar to Behaviour

> **The one-sentence version:** installing an upgrade changes the car's tuning fields, and those fields have *two
> readers* — the garage bar (which re-summarises them as a preview) and the vehicle sim (which reads them as the
> actual driving) — so the bar moving and the car getting faster are the same change, seen twice.

[← C69.3 — The three tuning bars](03-tuning-bars.md) · [Chapter 69 hub](C69-Performance-Upgrades-Tuning.md) ·
[Next: C69.5 — Reading upgrades in RE →](05-reading-upgrades.md)

---

## The full chain

Everything in this chapter connects into one chain, from the shop button to the car's behaviour:

```
buy part (C68.4)  ->  install in slot (C68.1)  ->  tuning fields change (Ch.13)
                                                        │
                                        ┌───────────────┴───────────────┐
                                        ▼                               ▼
                             garage bar re-summarises            vehicle sim reads
                             (C69.3, the preview)                (Ch.42, the reality)
```

The pivot is the **tuning fields** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)): the upgrade writes
them (via the part→vault mapping, [C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)), and *two
readers* consume them. This two-reader structure is the whole reason the garage can be trusted — the preview and the
reality read the *same* source.

## The two readers

The tuning fields feed two independent consumers, and it matters that they're the *same* fields:

- **The garage bar** ([C69.3](03-tuning-bars.md)) — reads the fields to *summarise* them into `TOPSPEED` /
  `ACCELERATION` / `HANDLING`. This runs in the front-end ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) when
  you're shopping.
- **The vehicle sim** ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) — reads
  the fields to *simulate* the car: the torque curve becomes acceleration ([C40.4](../C40-Eight-Mechanics/04-engine.md)),
  the grip becomes cornering ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)), the mass becomes
  inertia ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)). This runs on track when you drive.

Because both read the same fields, the bar is a *true* forecast: a part that raises the `TOPSPEED` bar raises the
field the sim uses for top speed, so the car really does go faster. There is no separate "performance number" the
designers could get wrong relative to the sim — the bar *is* a view of the sim's own inputs
([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)).

## No separate "apply"

A consequence worth stating plainly ([C68.1](../C68-Vehicles-Customisable-Object/01-car-object.md)): there is **no
apply-upgrades step**. Installing a part changes the fields, and *both readers just read the current fields* the next
time they run — the bar the next time the garage draws, the sim the next tick you drive. Nothing "recomputes the
car"; the car is *always* whatever its current fields say. This is the same live-state discipline as the HUD
([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) and the collision world's residency
([Chapter 63](../C63-Collision-World/C63-Collision-World.md)): state is read where it's needed, never cached and
re-applied. It's why you can install a part and *immediately* see the bar move and feel the car change — there's no
staging, just the current configuration being read.

## Why this design

The upgrade→bar→behaviour chain is a small masterclass in *single source of truth*:

- **One source** — the tuning fields ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
- **Written once** — by installing a part ([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)).
- **Read by two** — the bar (preview) and the sim (reality), each live.

The alternative — storing a separate "top speed rating" *and* the sim's top-speed field — would let them disagree
(the classic bug where the menu says one thing and the game does another). By deriving both the bar and the driving
from the one field set, MW makes the garage *honest*: what you see is what you get, because they're literally the
same numbers. Reading this chain is understanding why performance customization *feels* trustworthy — the preview
can't lie about the drive.

## RE implications

- **The full chain** — buy → install → tuning fields → {bar summary, sim} — pivoting on the tuning fields
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
- **Two readers, one source** — the garage bar ([C69.3](03-tuning-bars.md)) and the sim
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) read the *same* fields.
- **No separate apply** — both readers read the current fields live; the car is always its current configuration.
- **Single source of truth** — the bar can't lie about the drive, because they're the same numbers.

---

### Key takeaways

- Installing an upgrade changes the **tuning fields** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)),
  and those fields have **two readers**: the **garage bar** (summary/preview, [C69.3](03-tuning-bars.md)) and the
  **vehicle sim** (the actual drive, [Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).
- Because both read the **same fields**, the bar is a **faithful forecast** — a part that raises the `TOPSPEED` bar
  raises the field the sim uses for top speed, so the car really goes faster.
- There is **no "apply upgrades" step** — both readers read the current fields **live** (the bar when the garage
  draws, the sim each tick), so an install is felt immediately — the same live-state discipline as the HUD.
- The design is a **single source of truth**: one field set, written by installing, read by preview and reality — so
  the garage is **honest** (the preview can't disagree with the drive).
- This closes the cars-performance chain: **catalog** ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md))
  → **classes/tiers/ratings/bars** (this chapter) → **the sim**
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).

**Continue:** [C69.5 — Reading upgrades in RE](05-reading-upgrades.md) · [Chapter 69 hub](C69-Performance-Upgrades-Tuning.md)
