# C61.3 — Spawning

> **The one-sentence version:** traffic comes in two kinds — moving (`SpawnTraffic`, cars driving the roads) and
> parked (`AIParkedCarSpawner`, cars at the curb) — spawned by a shared `Spawner` machinery that also places cops
> (`SpawnCop`) and other actors (`SpawnCharacter`, `SpawnSmackable`).

[← C61.2 — Traffic density](02-traffic-density.md) · [Chapter 61 hub](C61-Traffic-Ambient.md) ·
[Next: C61.4 — Traffic behaviour →](04-traffic-behavior.md)

---

## Two kinds of traffic

The city has two kinds of ambient car, with two verified spawners:

- **Moving traffic** (`SpawnTraffic`) — cars *driving* the roads ([C61.4](04-traffic-behavior.md)), spawned into
  the ring ([C61.2](02-traffic-density.md)) on the road network ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)),
  lane-following at `TrafficSpeed`.
- **Parked cars** (`AIParkedCarSpawner`) — cars *stationary* at the curb, in lots, along the streets. They dress
  the world (a city has parked cars everywhere) and can be *smacked* into
  ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) — a parked car is an obstacle/smackable.

So the ambient population is *moving cars* (traffic you weave through) + *parked cars* (scenery you can hit). Both
are spawned into your vicinity ([C61.2](02-traffic-density.md)) — the moving ones on the roads, the parked ones at
their curb positions. Together they make the streets feel *used*: cars going places, cars left places.

> ✅ *Verified:* `SpawnTraffic` and `AIParkedCarSpawner` are present in `speed.exe`, alongside the general
> `Spawner`, `SpawnMode`, and the other spawn verbs (`SpawnCop`, `SpawnCharacter`, `SpawnSmackable`,
> `SpawnFragment`, `SpawnExplosion`).

## The shared Spawner machinery

The spawn verbs ([above](#two-kinds-of-traffic)) reveal a *shared* `Spawner` machinery — one system that places
*many kinds* of actor ([C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)):

| Verb | Spawns |
|---|---|
| `SpawnTraffic` | moving civilian cars ([C61.4](04-traffic-behavior.md)) |
| `AIParkedCarSpawner` | parked cars |
| `SpawnCop` | cop cars ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) |
| `SpawnCharacter` | characters (cutscene/bust actors, [Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)) |
| `SpawnSmackable` | knock-over props ([C43.5](../C43-Collision-Contacts/05-smackables.md)) |
| `SpawnFragment`/`SpawnExplosion` | debris / effects ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)) |

So there's *one* spawn system that populates the world with *whatever* — traffic, cops, props, debris. Each spawn
verb is a use of the shared `Spawner` ([C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)) machinery,
which finds a valid spawn point (on the road, at a curb, off-screen) and constructs the actor via the factory
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)). This is the engine's *composition*
economy again ([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)): one spawn mechanism, many actor types.
The traffic manager ([C61.1](01-traffic-system.md)) uses `SpawnTraffic`; the cop manager
([C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)) uses `SpawnCop` — both driving the same underlying
spawner.

## Spawning is off-screen and validated

Spawning ([above](#the-shared-spawner-machinery)) follows the same rules as cop spawning
([C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)):

- **Off-screen** — cars spawn *out of view* ([C61.2](02-traffic-density.md)) so you never see them appear — ahead,
  around corners, over rises.
- **On valid positions** — moving traffic spawns *on the road* ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md))
  (a valid lane), parked cars *at curb* positions — the spawner validates the placement.
- **Respecting the pool** — spawns come from the traffic pool ([C61.2](02-traffic-density.md),
  [Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)); if the pool is full (at density), no new spawn
  until one despawns.

So spawning is *careful*: it places believable cars in valid, unseen positions, bounded by the pool. This is what
makes the traffic *seamless* — you never catch a car materialising, and they're always where cars should be (on
roads, at curbs). The validation (on-road, off-screen, pooled) is the same production care as the cop spawner
([C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)) and the streaming budgets
([C38.6](../C38-Resource-Streaming-Residency/06-blocking-budgets.md)) — a shipped open world must populate itself
*invisibly*, and the spawn system is built to.

## RE implications

- **Two kinds of traffic** — moving (`SpawnTraffic`, roads) and parked (`AIParkedCarSpawner`, curbs).
- **A shared `Spawner`** places all actors — traffic, cops, characters, smackables, debris — one mechanism, many
  types.
- **Off-screen, validated, pooled** — cars spawn out of view, on valid positions, bounded by the pool.
- **Seamless population** — you never see a car appear; they're always where cars belong.

---

### Key takeaways

- Ambient traffic is **two kinds**: **moving** (`SpawnTraffic` — cars driving the roads) and **parked**
  (`AIParkedCarSpawner` — cars at curbs/lots, hittable smackables).
- Both use a **shared `Spawner` machinery** that places *all* actors — traffic, cops (`SpawnCop`), characters
  (`SpawnCharacter`), props (`SpawnSmackable`), debris — one mechanism, many types (the composition economy).
- Spawning is **off-screen** (never seen appearing), **on valid positions** (on-road for moving, at-curb for
  parked), and **pool-bounded** (from the traffic pool at density).
- This makes population **seamless** — you never catch a car materialising, and they're always where cars belong.
- The spawn validation is the same **production care** as the cop spawner and streaming budgets — a shipped open
  world populates itself **invisibly**.

**Continue:** [C61.4 — Traffic behaviour](04-traffic-behavior.md) · [Chapter 61 hub](C61-Traffic-Ambient.md)
