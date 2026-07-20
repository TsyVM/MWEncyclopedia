# C41.7 — Reading Physics in RE

> **The one-sentence version:** navigate the physics by three anchors — the class-name strings in `.rdata`, the
> class-name reflection hashes as vault keys, and the byte-verified functions (`Physics_Base::ctor`,
> `Physics::Simulate`, `IntegrateMotion`, `Math::Sqrt`) — reading the substrate from names, hashes, and opcodes.

[← C41.6 — Vehicle types](06-vehicle-types.md) · [Chapter 41 hub](C41-Physics-RigidBody.md) ·
[Next: Chapter 42 — Suspension, Tyres & Drivetrain →](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)

---

## Three kinds of anchor

The physics system is unusually legible because it's verifiable three independent ways:

- **Strings** ([C41.1](01-rigidbody-tree.md), [C41.6](06-vehicle-types.md)) — the class names (`RigidBody`,
  `RBVehicle`, `RBCop`…) and vehicle types (`BIKE`, `BOAT`…) in `.rdata`. Grep the executable, find the classes.
- **Hashes** ([C41.3](03-hash-unification.md)) — the class-name reflection hashes as keys in `attributes.bin`
  (`rh("EngineRacer")=0xB2809518` ×4). Compute a name's hash, find its vault data.
- **Bytes** ([C41.4](04-simulate-thiscall.md)–[C41.5](05-integrate-math.md)) — the physics functions decoded
  (`Physics::Simulate 0x6BB4D0`, `IntegrateMotion 0x6BA510`, `Math::Sqrt 0x5C5E80`, `Physics_Base::ctor
  0x6B9920`). Read the opcodes, confirm the behaviour.

Three anchors, three verification methods — strings, hashes, opcodes — converging on the same system. This is why
the physics chapter is among the most firmly grounded in the book.

## The RE workflow

Reading the physics:

1. **Find the classes** — grep `speed.exe` for `RigidBody`, `RB*`, and the vehicle types
   ([C41.1](01-rigidbody-tree.md), [C41.6](06-vehicle-types.md)); recover the class tree.
2. **Find their data** — hash the class/spec names ([C41.3](03-hash-unification.md)) and look the hashes up in
   `attributes.bin`; recover each class's parameters.
3. **Find the code** — the sim pipeline ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) from
   `FrameTick`'s sim driver down to `Physics::Simulate` and `IntegrateMotion`; confirm by the verified bytes
   ([C41.4](04-simulate-thiscall.md)–[C41.5](05-integrate-math.md)).
4. **Read a body** — in a dump, the transform `[this+0xF0]` and part array `[this+0xEC]`
   ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)); the vtable `[this+0]` identifies the class
   ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).

The output is the whole physics picture: the class tree, its data, its code, and a live body's state.

## Identifying a body's class in a dump

A body's **class** is identifiable from its vtable pointer ([C41.4](04-simulate-thiscall.md)): `[this+0]` is the
vtable, and each class ([C41.1](01-rigidbody-tree.md)) has a distinct vtable at a fixed address. So:

- **Read `[this+0]`** to get the vtable address.
- **Match it** to the known class vtables (`RBVehicle`'s vs. `RBCop`'s vs. `SimpleRigidBody`'s) — the vtable
  address *is* the class identity ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **The `+0x4C` slot** ([C41.4](04-simulate-thiscall.md)) in that vtable is the body's simulate virtual — reading
  it tells you which simulate behaviour this body runs.

So in a memory dump, "is this a cop or a player car?" is answered by the vtable pointer — `RBCop`'s vtable means a
cop, `RBVehicle`'s means a player/AI car. The class tree ([C41.1](01-rigidbody-tree.md)) is not just a source-code
abstraction; it's readable in the running game's memory, one vtable pointer per body.

## Verifying a physics claim

Every physics claim in this book reduces to one of the three anchor checks:

- **"Is there an `RBCop` class?"** — grep `speed.exe`: yes, at `0x4ADD6C` ([C41.1](01-rigidbody-tree.md)). ✅
- **"Is the engine spec keyed by name?"** — `rh("EngineRacer")=0xB2809518`, ×4 in `attributes.bin`
  ([C41.3](03-hash-unification.md)). ✅
- **"Is `0x5C5E80` really a square root?"** — `D9 44 24 04 D9 FA C3` = `fld;fsqrt;ret`
  ([C41.5](05-integrate-math.md)). ✅
- **"Is `Simulate` a `__thiscall`?"** — `mov esi,ecx` at `0x6BB4D0` ([C41.4](04-simulate-thiscall.md)). ✅

A string offset, a hash count, or an opcode sequence — each claim is checkable, and each was checked. This is the
verification-first discipline ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) at
its strongest, because the physics offers all three kinds of evidence at once.

## Physics grounds the vehicle

With the rigid-body substrate decoded, the rest of the vehicle chapters stand on solid ground: the **mechanics**
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) are behaviours on these bodies, **suspension/tyres**
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) are the wheel forces the
body integrates, **collision** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) is the contact
update on the body, **damage** ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)) deforms the
body, and **AI** ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) drives it. The `RigidBody` tree,
`Physics_Base`, and the integrator are the foundation those chapters build on — verified in strings, hashes, and
bytes.

## RE implications

- **Three anchors** — strings (`.rdata` class names), hashes (vault keys), bytes (physics functions) — verify the
  physics three ways.
- **The RE workflow** — classes (grep) → data (hash) → code (bytes) → live body (dump).
- **A body's class is its vtable pointer** — `[this+0]` identifies `RBVehicle` vs. `RBCop` vs. `SimpleRigidBody`
  in a dump.
- **Every claim reduces to a check** — a string offset, a hash count, or an opcode sequence.

---

### Key takeaways

- The physics is verifiable **three independent ways**: **strings** (class names in `.rdata`), **hashes**
  (class-name reflection hashes as vault keys), and **bytes** (decoded physics functions).
- The RE workflow: grep the **classes** → hash to find their **data** → trace the **code** → read a live **body**.
- A body's **class is its vtable pointer** (`[this+0]`) — `RBCop` vs. `RBVehicle` is readable in a memory dump.
- Every physics claim reduces to a **string offset, hash count, or opcode** — and each was checked.
- The rigid-body substrate **grounds the whole vehicle** — mechanics, tyres, collision, damage, and AI build on
  it.

**Next:** [Chapter 42 — Suspension, Tyres & Drivetrain](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md):
the wheel forces the rigid body integrates.

**Sources:** `speed.exe` (verified: class strings `RigidBody`/`SimpleRigidBody`/`RBSmackable`/`RBVehicle`/`RBCop`/
`RBTrailer` and vehicle-type tokens; `Physics_Base::ctor 0x6B9920`, `Physics::Simulate 0x6BB4D0`, `IntegrateMotion
0x6BA510`, `Math::Sqrt 0x5C5E80`); `GLOBAL/attributes.bin` (verified: class-name reflection hashes as vault keys —
`EngineRacer` ×4, `DamageRacer` ×3, `RigidBody`/`RBVehicle`/`RBCop`/`RBTrailer` ×1).
