# Chapter 41 — Physics & Rigid-Body Dynamics

> **Goal of this chapter:** decode the rigid-body class family Most Wanted's vehicles are built on — the
> `RigidBody` tree (`SimpleRigidBody`, `RBSmackable`, `RBVehicle`, `RBCop`, `RBTrailer`), the multi-interface
> `Physics_Base` they descend from, and the reflection-hash unification that keys their vault specs — all
> recovered from real strings and bytes in `speed.exe`.

Every moving thing in Most Wanted — the player's car, cops, traffic, the props you smash — is a **rigid body**.
This chapter is the physics substrate the eight mechanics ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md))
run on and the sim pipeline ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) ticks: the class
hierarchy of bodies, the interface-based `Physics_Base` they share, and the math that integrates them. It is one
of the most thoroughly *verified* chapters in this book — the class names are in the executable, and the physics
functions are decoded to the byte.

> **Verified against the executable and vault.** The rigid-body class family exists as a **string table in
> `speed.exe` `.rdata`**: `SimpleRigidBody` (file-offset `0x4AA4CC`), `RBSmackable` (`0x4AAA04`), `RigidBody`
> (`0x4AAA10`), `RBVehicle` (`0x4ADD54`), `RBTrailer` (`0x4ADD60`), `RBCop` (`0x4ADD6C`). Their **reflection
> hashes key the vault**: in `GLOBAL/attributes.bin`, `rh("RigidBody")=0xB7235989` (×1), `rh("RBVehicle")=0x3109B00F`
> (×1), `rh("RBCop")=0xBE725CD4` (×1), `rh("RBTrailer")=0x87FF41E0` (×1), `rh("EngineRacer")=0xB2809518` (×4). The
> physics functions are byte-verified: `Physics_Base::ctor 0x6B9920` = `6A FF 68 12 DF 87 00` (SEH prologue);
> `Physics::Simulate 0x6BB4D0` = `56 8B F1 8B 06 57 FF 50 4C` (`__thiscall`, `call [eax+0x4C]`); `IntegrateMotion
> 0x6BA510` = `81 EC 30 05 00 00` (`sub esp,0x530`); `Math::Sqrt 0x5C5E80` = `D9 44 24 04 D9 FA C3`
> (`fld [esp+4]; fsqrt; ret`). ImageBase `0x400000`; code file-offset = VA − `0x400000`.

---

## The rigid-body family

| Class | .rdata offset | Role |
|---|---|---|
| `RigidBody` | `0x4AAA10` | the base physics body — mass, velocity, transform, integrate |
| `SimpleRigidBody` | `0x4AA4CC` | a lightweight body (simple props, debris) |
| `RBSmackable` | `0x4AAA04` | a "smackable" world object — knock-over props |
| `RBVehicle` | `0x4ADD54` | a full vehicle — the player and AI cars |
| `RBCop` | `0x4ADD6C` | a cop vehicle — `RBVehicle` with pursuit behaviour |
| `RBTrailer` | `0x4ADD60` | a towed trailer body |

---

## Deep-dive pages

- [C41.1 — The RigidBody class tree](01-rigidbody-tree.md): the verified family and its hierarchy.
- [C41.2 — Physics_Base & its interfaces](02-physics-base.md): the multi-interface base (ctor `0x6B9920`).
- [C41.3 — The hash unification](03-hash-unification.md): class names → reflection hash → vault specs.
- [C41.4 — Physics::Simulate, byte by byte](04-simulate-thiscall.md): the `__thiscall` and `[eax+0x4C]` dispatch.
- [C41.5 — IntegrateMotion & the math](05-integrate-math.md): the integrator and `Math::Sqrt` (`fld;fsqrt;ret`).
- [C41.6 — Vehicle types: the engine's breadth](06-vehicle-types.md): BIKE/BOAT/CHOPPER and the shared engine.
- [C41.7 — Reading physics in RE](07-reading-physics.md): navigating the physics by strings, hashes, and bytes.

---

## 41.1 The RigidBody tree

The physics bodies form a **class tree** rooted at `RigidBody` ([C41.1](01-rigidbody-tree.md)), all present as
strings in `speed.exe`. `RigidBody` is the base (mass, velocity, transform, integrate); `SimpleRigidBody` and
`RBSmackable` are lighter bodies (debris, knock-over props); `RBVehicle` is the full car, and `RBCop` is a cop
car (an `RBVehicle` specialised for pursuit). The tree is the class system
([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) applied to physics — one base body, many
specialisations.

## 41.2 Physics_Base

Under the whole tree is **`Physics_Base`**, whose constructor at `0x6B9920` (`6A FF 68 12 DF 87 00` — an SEH
prologue) **embeds interface sub-objects** ([C41.2](02-physics-base.md)). A physics body implements *multiple
interfaces* — it can be seen as a simulate-able thing, a collide-able thing, and more — so the sim, the collision
world ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)), and the mechanics
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) each hold the body through the interface they care
about. This multiple-inheritance-of-interfaces is why the body is a hub the whole vehicle system plugs into.

## 41.3 The hash unification

The class names double as **vault keys** through the reflection hash
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)). The same Jenkins-derived hash that keys vault
fields also hashes the class names, and those hashes appear in `attributes.bin` as keys
([C41.3](03-hash-unification.md)): `rh("EngineRacer")=0xB2809518` appears **4 times**, `rh("DamageRacer")=0x6AE5E09C`
**3 times**, `rh("RigidBody")=0xB7235989` once. So a car's per-class specs (its engine spec, damage spec,
suspension spec) are looked up by hashing the spec's *class name*. This is the unification: **one hash namespace
spans runtime class names and vault data** — the names in the exe and the keys in the vault are the same values.

## 41.4 Physics::Simulate

`Physics::Simulate` (`0x6BB4D0`) is byte-verified as a `__thiscall` that immediately dispatches a virtual:
`56 8B F1 8B 06 57 FF 50 4C` = `push esi; mov esi,ecx; mov eax,[esi]; push edi; call [eax+0x4C]`
([C41.4](04-simulate-thiscall.md)). The `mov esi,ecx` takes the body in `ECX`; the `call [eax+0x4C]` dispatches
the body's virtual at **vtable offset `+0x4C`** — the polymorphic entry that lets each body type
([C41.1](01-rigidbody-tree.md)) run its own simulate behaviour through one pipeline.

## 41.5 IntegrateMotion & the math

`IntegrateMotion` (`0x6BA510`, `81 EC 30 05 00 00` — `sub esp,0x530`) is the integrator
([C41.5](05-integrate-math.md)), and its speed computation calls **`Math::Sqrt` (`0x5C5E80`)**, which is verified
to be *literally* `D9 44 24 04 D9 FA C3` = `fld dword [esp+4]; fsqrt; ret` — an FPU square root. This is the kind
of unambiguous verification the book is built on: a function that loads a float, runs the `fsqrt` instruction,
and returns *is* a square root, beyond doubt.

## 41.6 The engine's breadth

The .rdata strings reveal the physics engine is far broader than what Most Wanted ships: alongside the car
classes sit vehicle-type tokens **`BIKE`, `BOAT`, `CHOPPER`, `SUBMARINE`, `SNOWMOBILE`, `HOVER`, `PLANE`** and
spec classes like `SimpleChopper` and `DamageHeli` ([C41.6](06-vehicle-types.md)). Most Wanted uses cars (and
bikes in a limited way), but the underlying EA Black Box physics engine — shared across titles — supports boats,
planes, helicopters, and more. The class tree is a window onto the engine's wider lineage, an industry artifact
of a reused codebase.

---

### Key takeaways

- Most Wanted's moving objects are **rigid bodies** — a verified class tree (`RigidBody`, `SimpleRigidBody`,
  `RBSmackable`, `RBVehicle`, `RBCop`, `RBTrailer`) present as strings in `speed.exe`.
- All descend from **`Physics_Base`** (ctor `0x6B9920`), which **embeds interface sub-objects** — one body, many
  interfaces (sim, collision, mechanics).
- The class names **key the vault** via the reflection hash — `rh("EngineRacer")=0xB2809518` (×4) etc. in
  `attributes.bin` — the **one-hash unification** of class names and vault data.
- The physics functions are **byte-verified**: `Physics::Simulate` (`__thiscall`, `call [eax+0x4C]`),
  `IntegrateMotion` (`sub esp,0x530`), and `Math::Sqrt` (`fld;fsqrt;ret` — unambiguously a square root).
- The class strings reveal the **engine's breadth** (BIKE/BOAT/CHOPPER/PLANE…) — a shared EA Black Box physics
  lineage beyond what MW ships.

**Next:** [Chapter 42 — Suspension, Tyres & Drivetrain](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md):
the wheel forces the rigid body integrates.
