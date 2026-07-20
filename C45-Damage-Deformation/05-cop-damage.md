# C45.5 — Cop Damage & the Bust

> **The one-sentence version:** `DamageCopCar` (vtable `0x008AD3F4`, 36 methods — the fewest) is the cop damage
> model, with cop-specific parts `DAMAGE_COP_LIGHTS` and `DAMAGE_COP_SPOILER` — cruisers take damage and get
> *disabled/totalled* rather than finely crumpled, the basis of the disable-the-cop mechanic.

[← C45.4 — Scaling & performance loss](04-scaling-performance.md) · [Chapter 45 hub](C45-Damage-Deformation.md) ·
[Next: C45.6 — Reading damage in RE →](06-reading-damage.md)

---

## The cop damage model

Cops use their own damage class — **`DamageCopCar`** (verified vtable `0x008AD3F4`, **36 methods** — the *fewest*
of the Damage family, [C45.1](01-damage-family.md)). The small method count is meaningful: a cop car doesn't need
the player's fine-grained crumple ([C45.3](03-deformation.md)) and handling-degradation detail. It needs a coarser
model built around one outcome — being **disabled**.

- **Takes damage** from the player ramming it, from crashes, from other cops.
- **Gets disabled/totalled** past a threshold — the cruiser is knocked out of the pursuit
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **Coarser detail** — 36 methods for "accumulate damage → disable," versus `DamageRacer`'s 98 for the full player
  experience.

So `DamageCopCar` is the "disable-oriented" damage model — enough to make cops destructible, without the expense of
modelling every cruiser's crumple like the player's car.

> ✅ *Verified:* `DamageCopCar` is a real vtable at `0x008AD3F4` with **36 methods** (the fewest of the Damage
> family); `rh("DamageCopCar")=0x1DF44901` ×1 in `attributes.bin`. The cop-specific damage parts
> `DAMAGE_COP_LIGHTS` and `DAMAGE_COP_SPOILER` are verified strings in `speed.exe`.

## The cop light bar

The verified cop-specific parts — **`DAMAGE_COP_LIGHTS`** (the light bar) and **`DAMAGE_COP_SPOILER`** — are what
make a cop car *a cop car* at the damage level ([C45.2](02-damage-zones.md)):

- **`DAMAGE_COP_LIGHTS`** — the roof light bar, a breakable ([C45.3](03-deformation.md)) part unique to cruisers.
  Knock it off and the cruiser visibly loses its lights — a satisfying, readable sign you've hit it hard.
- **`DAMAGE_COP_SPOILER`** — a cop spoiler part, likewise breakable.

These parts exist *because* cops are a distinct role: a player car has headlights and bumpers
([C45.2](02-damage-zones.md)); a cop additionally has a light bar. The damage system knows about the cop's extra
parts, so the light bar deforms and detaches like any breakable — but its meaning is cop-specific (a disabled
cruiser). This is the damage-level counterpart to `RBCop` extending `RBVehicle`
([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)): the cop is the car plus cop-specific bits, including its
damageable light bar.

## Disabling cops: the bust in reverse

The cop damage model underpins a core pursuit interaction ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)):
**disabling cop cars.** Where the cops try to *bust* you ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)),
you can fight back by *disabling* them:

- **Ram a cruiser** enough ([C43.4](../C43-Collision-Contacts/04-reactions.md)) — into a wall, off the road, or
  head-on — and `DamageCopCar` accumulates damage until the cruiser is **totalled** and drops out of the pursuit.
- **Use the world** — pursuit breakers ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) that drop objects
  on cops, or leading them into hazards, disable multiple cruisers at once.
- **The pursuit thins** — each disabled cop is one fewer chasing you, easing the heat
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).

So `DamageCopCar` is the mechanical basis of the "fight back" side of a pursuit — the cops aren't invincible; their
damage model lets you knock them out. This two-way destructibility (they damage you, [C45.4](04-scaling-performance.md);
you disable them) is central to Most Wanted's pursuit feel: a running battle, not just a chase. The 36-method model
is tuned for exactly that — durable enough to be a threat, destructible enough to be a target.

> 🟡 *Reasoned:* the disable-the-cruiser interaction built on `DamageCopCar` is the natural reading of the verified
> cop damage class, its cop-specific breakable parts, and the pursuit design
> ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)); the exact disable thresholds are vault tunables. The
> class, method count, and cop parts are verified.

## Why a separate cop damage class

Giving cops their own damage class (rather than reusing `DamageRacer`) follows the per-role fidelity pattern
([C45.1](01-damage-family.md)) and adds cop specifics:

- **Different fidelity.** Cops need "disable," not full crumple — 36 methods, not 98 ([C45.1](01-damage-family.md)).
- **Cop-specific parts.** The light bar and spoiler ([above](#the-cop-light-bar)) are cop-only breakables the
  player car doesn't have.
- **Different outcome.** A totalled cop *leaves the pursuit* ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md))
  — a game-state change, not just a wrecked-car visual. The cop damage model ties into pursuit logic.

So `DamageCopCar` is where the damage system meets the pursuit system: it models cops as destructible pursuit
units with their own parts and their own "disabled" outcome. It's a small class (36 methods) with an outsized
gameplay role — the reason you can turn and fight the cops chasing you.

## RE implications

- **`DamageCopCar` (36 methods, `0x008AD3F4`)** is the cop damage model — coarse, disable-oriented (fewest methods
  of the family).
- **Cop-specific parts** — `DAMAGE_COP_LIGHTS` (light bar) and `DAMAGE_COP_SPOILER` — verified breakables unique to
  cruisers.
- **Disabling cops** — ram/hazard a cruiser until `DamageCopCar` totals it and it leaves the pursuit
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **A separate class** for cop fidelity, cop parts, and the "disabled → leaves pursuit" outcome.

---

### Key takeaways

- **`DamageCopCar`** (verified vtable `0x008AD3F4`, **36 methods** — fewest of the family) is the cop damage model:
  coarse and **disable-oriented**, not finely crumpled.
- **Cop-specific breakable parts** — `DAMAGE_COP_LIGHTS` (the light bar) and `DAMAGE_COP_SPOILER` — are verified,
  the damage-level mark of a cop.
- The model underpins **disabling cops** — ram or hazard a cruiser until it's totalled and drops out of the
  pursuit.
- This makes pursuit a **two-way battle** — cops damage you ([C45.4](04-scaling-performance.md)); you disable them
  — central to MW's feel.
- A **separate cop damage class** captures the different fidelity, the cop-only parts, and the "disabled → leaves
  pursuit" game-state outcome.

**Continue:** [C45.6 — Reading damage in RE](06-reading-damage.md) · [Chapter 45 hub](C45-Damage-Deformation.md)
