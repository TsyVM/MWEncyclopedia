# C64.3 — One-Shot Effects

> **The one-sentence version:** `World_OneShotEffect` handles fire-and-forget effects — one-time visuals
> (explosions, sparks, bursts) that the world spawns, plays once, and forgets — the pattern for transient events
> with no owner to manage them.

[← C64.2 — The active-body list](02-active-body-list.md) · [Chapter 64 hub](C64-World-Update.md) ·
[Next: C64.4 — World animations →](04-world-animations.md)

---

## Fire-and-forget

**`World_OneShotEffect`** ([C64.1](01-world-tick.md)) handles **one-shot effects** — effects that play *once* and
*self-complete*, with no persistent owner:

- **An explosion** ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)) — a burst of fire and debris
  that flares and fades.
- **A spark burst** — from a scrape ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) or a
  collision.
- **A nitrous flash**, a smackable's debris scatter ([C43.5](../C43-Collision-Contacts/05-smackables.md)) — a
  one-time visual.

These are **fire-and-forget**: the world *spawns* the effect at a position, it *plays* (its particles emit and
fade, [C52.2](../C52-Effects-Particles/02-emitters-particles.md)), and it *finishes* and is *forgotten* — no owner
holds onto it. `World_OneShotEffect` is the world's mechanism for these — a list of playing one-shots, walked each
frame to advance them, removing the finished ones.

> ✅ *Verified:* `World_OneShotEffect` and `OneShotEffect` are present in `speed.exe` — the fire-and-forget effect
> mechanism.

## One-shot vs. persistent effects

One-shots ([above](#fire-and-forget)) contrast with the *persistent* effects of the entity connectors
([C52.4](../C52-Effects-Particles/04-entity-effects.md)):

| | Persistent effect ([C52.4](../C52-Effects-Particles/04-entity-effects.md)) | One-shot effect (this page) |
|---|---|---|
| Owner | an entity (`EffectsCar`) | none (the world) |
| Lifetime | while the entity is active | plays once, self-completes |
| Example | a car's continuous exhaust | an explosion |
| Managed by | the entity's connector | `World_OneShotEffect` |

So there are *two* effect lifecycles: **persistent** (owned by an entity, playing while it's active — a car's
exhaust follows the car) and **one-shot** (unowned, played once by the world — an explosion happens and is gone).
The distinction matters because they're *managed differently*: a persistent effect is tied to its entity (spawned
and stopped with it, [C52.4](../C52-Effects-Particles/04-entity-effects.md)); a one-shot is *detached* — spawned
into the world and self-terminating. `World_OneShotEffect` exists precisely for the *unowned* case — a transient
event that no entity should have to manage.

## Why fire-and-forget

Handling transient effects as *world-owned fire-and-forget* ([above](#one-shot-vs-persistent-effects)) is a clean
solution to a real problem:

- **No owner needed.** An explosion at a crash site ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md))
  has no natural *owner* — the cars involved may drive away or be destroyed. A one-shot needs no owner; the world
  plays it. This *decouples* the effect from any entity's lifetime.
- **Self-terminating.** A one-shot *completes itself* (the explosion fades) and removes itself from the list — no
  cleanup by anyone. This avoids leaks and lifetime bugs (a persistent effect whose owner dies could orphan it; a
  one-shot can't).
- **Bounded.** One-shots come from the effect pools ([C52.2](../C52-Effects-Particles/02-emitters-particles.md)),
  so their count is bounded ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — a burst of many
  crashes plays many one-shots, capped by the pool.

So `World_OneShotEffect` is the *transient-event* effect pattern — for the many one-time visuals (explosions,
bursts, scatters) that happen *at* the world (not *on* an entity) and should self-clean. It's the world-level
counterpart to the entity effects ([C52.4](../C52-Effects-Particles/04-entity-effects.md)): entity effects are
*attached* (a car's smoke), one-shots are *emitted into the world* (a crash's fireball). Both use the particle
system ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)); they differ in *ownership and lifetime*.
Reading `World_OneShotEffect` completes the effect picture — persistent (entity-owned) plus one-shot
(world-emitted).

## RE implications

- **`World_OneShotEffect`** handles fire-and-forget effects — spawned, played once, self-completed, forgotten.
- **One-shot vs. persistent** — unowned/transient (explosions) vs. entity-owned/continuous (a car's exhaust).
- **Fire-and-forget** solves the *no-owner* case — transient events at the world, self-terminating, bounded by
  pools.
- **World-emitted vs. entity-attached** — one-shots complete the effect picture with the entity effects
  ([C52.4](../C52-Effects-Particles/04-entity-effects.md)).

---

### Key takeaways

- **`World_OneShotEffect`** handles **fire-and-forget** effects — one-time visuals (explosions, sparks, debris
  scatters) the world **spawns, plays once, and forgets** — walked and pruned each frame.
- One-shots contrast with **persistent** effects ([C52.4](../C52-Effects-Particles/04-entity-effects.md)) —
  **unowned/transient** (an explosion) vs. **entity-owned/continuous** (a car's exhaust) — managed differently.
- Fire-and-forget solves the **no-owner** case — a transient event at the world (a crash's fireball) with no entity
  to manage it — **self-terminating** (no leaks) and **pool-bounded**.
- One-shots are **emitted into the world**; entity effects are **attached to an entity** — both use the particle
  system, differing in **ownership and lifetime**.
- `World_OneShotEffect` **completes the effect picture** — persistent (entity-owned) + one-shot (world-emitted).

**Continue:** [C64.4 — World animations](04-world-animations.md) · [Chapter 64 hub](C64-World-Update.md)
