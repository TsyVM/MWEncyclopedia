# C41.6 — Vehicle Types: the Engine's Breadth

> **The one-sentence version:** the physics `.rdata` holds vehicle-type tokens far beyond cars — `BIKE`, `BOAT`,
> `CHOPPER`, `SUBMARINE`, `SNOWMOBILE`, `HOVER`, `PLANE`, plus spec classes like `SimpleChopper` and `DamageHeli`
> — a window onto the shared EA Black Box physics engine's wider lineage.

[← C41.5 — IntegrateMotion & the math](05-integrate-math.md) · [Chapter 41 hub](C41-Physics-RigidBody.md) ·
[Next: C41.7 — Reading physics in RE →](07-reading-physics.md)

---

## The tokens in .rdata

Right beside the rigid-body class names ([C41.1](01-rigidbody-tree.md)), the executable's `.rdata` holds a set of
**vehicle-type tokens** — a verified string cluster:

```
0x4AAA20  SUBMARINE
0x4AAA2C  CHOPPER
0x4AAA34  BIKE
0x4AAA3C  BOAT
0x4AAA44  SNOWMOBILE
0x4AAA50  HOVER
0x4AAA58  PLANE
```

And nearby, spec classes for non-car vehicles: `SimpleChopper` (`0x4ADDFC`), `DamageHeli` (`0x4ADDAC`). Most
Wanted is a game about *cars* (and, marginally, motorbikes) — yet the physics engine names submarines,
helicopters, boats, snowmobiles, hovercraft, and planes.

> ✅ *Verified:* the vehicle-type tokens `SUBMARINE`, `CHOPPER`, `BIKE`, `BOAT`, `SNOWMOBILE`, `HOVER`, `PLANE`
> are present as strings in `speed.exe` `.rdata` (offsets `0x4AAA20`–`0x4AAA58`), alongside `SimpleChopper`
> (`0x4ADDFC`) and `DamageHeli` (`0x4ADDAC`).

## A shared engine's lineage

These tokens are an **industry artifact**: they reveal that Most Wanted's physics is built on a **shared,
general-purpose vehicle engine** — the EA Black Box / EAGL technology reused across titles. That engine supports
many vehicle classes, and its full vocabulary is compiled into `speed.exe` even though Most Wanted only *uses* the
car (and bike) paths. The other tokens are dormant capabilities of the codebase:

- **`BOAT`, `SUBMARINE`, `HOVER`** — watercraft physics (buoyancy, water drag) — used in EA titles with water
  gameplay, not in MW's street racing.
- **`CHOPPER`, `PLANE`** (and `SimpleChopper`, `DamageHeli`) — aircraft physics — MW's police helicopter
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) is a *scripted* chopper, and
  these classes are the physics substrate for flying vehicles in the engine lineage.
- **`SNOWMOBILE`, `BIKE`** — two-wheel/track vehicles — the engine's lean-and-balance handling.

So the class strings are a fossil record of the engine's breadth. Reading them tells you not just what Most
Wanted is, but what its *engine* can do — a codebase built for a range of vehicles, deployed here for cars.

> 🟡 *Reasoned:* that these tokens reflect a shared EA Black Box vehicle engine reused across titles is inferred
> from the tokens' presence and the known EAGL lineage, consistent with the verified strings; which specific
> other titles used which tokens is outside this executable. The tokens themselves are verified present.

## The police helicopter

The one non-car vehicle Most Wanted actually features is the **police helicopter** — and the `CHOPPER` /
`SimpleChopper` / `DamageHeli` classes are its physics lineage. In MW the chopper is largely *scripted* (it
follows the player on a controlled path rather than being a fully free-flying `RBVehicle`), but its damage
(`DamageHeli`) and its class identity (`SimpleChopper`) draw on the engine's aircraft support. So even MW's one
flying vehicle traces back to the shared engine's broader vehicle set. This is the pursuit chopper
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) at the class level.

## Why dormant classes remain

That unused vehicle classes are compiled into the shipping executable is normal and instructive:

- **Shared codebase.** The physics engine is a library shared across projects; a given game links the whole
  library, using the subset it needs. Stripping unused classes would fork the shared code — not worth it.
- **Data-driven activation.** A vehicle type is *activated by data* (a car's spec referencing the class,
  [C41.3](03-hash-unification.md)) — the code is always present, the data decides what's used. MW's data
  references cars and the scripted chopper; it doesn't reference boats, so the boat code lies dormant.
- **Engineering economy.** Reusing one vehicle engine across many games — cars here, boats and planes elsewhere —
  is the economy of a large studio's shared technology. The class strings are the evidence of that reuse.

So the vehicle-type tokens are a lesson in game-engine economics: a shipped game is a *configuration* of a larger
engine, and the executable carries the engine's full vocabulary even when the game speaks only part of it.
Reverse-engineering the strings reveals the engine behind the game.

## RE implications

- **The physics `.rdata` names many vehicle types** — `SUBMARINE`, `CHOPPER`, `BIKE`, `BOAT`, `SNOWMOBILE`,
  `HOVER`, `PLANE` (verified).
- **They reveal a shared EA Black Box vehicle engine** — MW is a car-configuration of a broader engine.
- **MW's one flying vehicle** (the pursuit chopper) draws on `CHOPPER`/`SimpleChopper`/`DamageHeli`.
- **Dormant classes are normal** — the code ships whole; the data activates the subset used.

---

### Key takeaways

- Beyond cars, the physics `.rdata` names **`SUBMARINE`, `CHOPPER`, `BIKE`, `BOAT`, `SNOWMOBILE`, `HOVER`,
  `PLANE`** (plus `SimpleChopper`, `DamageHeli`) — **verified strings**.
- These reveal Most Wanted is a **car-configuration of a shared, general-purpose EA Black Box vehicle engine**.
- MW's **police helicopter** is the one flying vehicle, drawing on the `CHOPPER`/`DamageHeli` lineage (largely
  scripted).
- **Dormant vehicle classes remain** in the executable because the physics engine is a **shared library** — code
  ships whole, data activates the subset.
- The tokens are a lesson in **engine economics** — a shipped game is a configuration of a larger engine, its
  full vocabulary visible in the strings.

**Continue:** [C41.7 — Reading physics in RE](07-reading-physics.md) · [Chapter 41 hub](C41-Physics-RigidBody.md)
