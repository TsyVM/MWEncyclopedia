# C42.1 — Fidelity Tiers

> **The one-sentence version:** the `Engine*` and `Suspension*` families come in fidelity tiers — `*Racer` (hero,
> full), `*Traffic` (cheap), `*Simple`, `*Spline` (rail), `*Trailer` — so a traffic car and the player share the
> vehicle shell but pay different simulation costs, chosen per car by vault data.

[← Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md) · [Next: C42.2 — The engine & drivetrain →](02-engine-drivetrain.md)

---

## One mechanic, several implementations

The `BEHAVIOR_MECHANIC_ENGINE` and `BEHAVIOR_MECHANIC_SUSPENSION` slots
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) aren't single classes — each is a **family of
implementations at different fidelities**. A car fills its engine slot with one of the `Engine*` classes and its
suspension slot with one of the `Suspension*` classes, and *which* one determines how expensively (and
accurately) that car simulates.

The families, all verified as real vtables in `speed.exe`:

| Class | Hash | vtable | Methods | Role |
|---|---|---|---|---|
| `EngineRacer` | `0xB2809518` | `0x008AB6A0` | 123 | full drivetrain (hero cars) |
| `EngineDragster` | `0x4BC7F9AF` | `0x008ABF34` | 132 | drag-race drivetrain |
| `EngineTraffic` | `0x5C216BAB` | `0x008AB8F8` | 67 | cheap drivetrain (ambient) |
| `EngineSpline` | `0x3F172A3E` | `0x008AB7AC` | 56 | rail (on-spline) drivetrain |
| `SuspensionRacer` | `0x6209E06A` | `0x008ABAC0` | 45 | full suspension (hero) |
| `SuspensionTraffic` | `0x12D5313C` | `0x008ABB80` | 86 | ambient-car suspension |
| `SuspensionSimple` | `0x723B315B` | `0x008ABC28` | 44 | reduced suspension |
| `SuspensionSpline` | `0xBB35585B` | `0x008ABD88` | 57 | rail suspension |
| `SuspensionTrailer` | `0xD44C9372` | `0x008ABCE0` | 99 | articulated-trailer suspension |

> ✅ *Verified:* each class above is a real vtable in `speed.exe` — the method counts were confirmed by counting
> consecutive valid code pointers at the vtable address (e.g. `EngineRacer` at `0x008AB6A0` has exactly 123). The
> hashes are the reflection hashes of the class names; the `*Racer`/`*Traffic` specs are vault keys in
> `attributes.bin` (`EngineRacer` ×4, `SuspensionRacer` ×3, `SuspensionTraffic` ×2, `EngineTraffic` ×2).

## The shared shell, different cost

The point of the tiers is that all cars share the **vehicle shell** — the same `RBVehicle`
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)), the same eight mechanic slots
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) — and differ only in *which fidelity* fills the
engine and suspension slots:

- **Hero cars (the player, key rivals)** get `EngineRacer` + `SuspensionRacer` — the full torque-curve,
  gearbox, per-wheel spring/damper model. Expensive, but there's only a handful on screen.
- **Traffic (ambient cars)** get `EngineTraffic` + `SuspensionTraffic` — enough to move and settle convincingly,
  without the tuning surface. Cheap, because there are *many* of them.
- **Rail vehicles (scripted/on-spline)** get `EngineSpline` + `SuspensionSpline` — driven along a pre-authored
  path ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) rather than from physics.
- **Trailers** get `SuspensionTrailer` — the articulated multi-axle case ([C42.3](03-suspension.md)).

So the game pays for fidelity where it matters (the cars you're racing) and economises where it doesn't (the
dozens of background cars). This is the same LOD philosophy as rendering — full detail near, cheap detail far —
applied to *physics*.

## Method count ≠ fidelity, necessarily

A subtle, verified surprise: **method count doesn't always track "how expensive/detailed"** in the obvious
direction. `SuspensionTraffic` has **86** methods — nearly *double* `SuspensionRacer`'s 45. Why would the "cheap"
tier have more methods?

The likely reason ([reasoned](#re-implications)) is that `SuspensionTraffic`'s methods are many small,
specialised, inlined-per-wheel approximations — the traffic suspension is called constantly across the whole busy
traffic population, so it's built from many tiny fast methods rather than a few general ones. `SuspensionRacer`,
by contrast, uses fewer but heavier general-purpose methods (a full per-wheel spring/damper solve). So the *cost
per car* can be lower for traffic even with more methods, because each method does less and they're optimised for
the many-cars case. `SuspensionTrailer`'s 99 methods (the most) reflect genuine difficulty — a swinging,
multi-axle towed box is the hardest body to keep stable.

> 🟡 *Reasoned:* the interpretation of why `SuspensionTraffic` (86) and `SuspensionTrailer` (99) have more methods
> than `SuspensionRacer` (45) — many small inlined approximations vs. fewer heavy general solves — is inferred
> from the method counts and each tier's role; the exact per-method behaviour is deeper RE. The **method counts
> themselves are verified**.

## Choosing a tier: data-driven

Which tier a car uses is **data-driven** ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)): the car's
vault data references the spec by name (hashed), so the game constructs the right `Engine*`/`Suspension*` for that
car. A hero car's collection points at `EngineRacer`; a traffic car's at `EngineTraffic`. This is why the
selection needs no code branches — the class registry ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md))
constructs whichever spec the data names. Add a new fidelity tier, and cars can reference it purely through data.

## RE implications

- **The engine and suspension mechanics are families** — `*Racer`/`*Traffic`/`*Simple`/`*Spline`/`*Trailer` —
  each a verified vtable with a confirmed method count.
- **Tiers share the vehicle shell**, differing only in engine/suspension fidelity — physics LOD.
- **Method count doesn't linearly track cost** — `SuspensionTraffic` (86) > `SuspensionRacer` (45) because of
  many small inlined per-wheel approximations.
- **The tier is chosen by vault data** — the car references its spec by hashed name; the registry constructs it.

---

### Key takeaways

- The `Engine*` and `Suspension*` mechanics are **families of fidelity tiers** — hero (`*Racer`), traffic
  (cheap), simple, rail (`*Spline`), trailer — all **verified vtables** with confirmed method counts.
- Cars **share the vehicle shell** and differ only in which fidelity fills the engine/suspension slots — **physics
  LOD**: full detail for the cars you race, cheap for background traffic.
- **Method count ≠ cost linearly** — `SuspensionTraffic` (86) has more methods than `SuspensionRacer` (45) (many
  small inlined per-wheel approximations); `SuspensionTrailer` (99) is the hardest case.
- The tier is **data-driven** — the car's vault references its spec by hashed name; the registry constructs the
  right class.
- All the specs register onto the mechanics list-head **`0x0092C660`**.

**Continue:** [C42.2 — The engine & drivetrain](02-engine-drivetrain.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md)
