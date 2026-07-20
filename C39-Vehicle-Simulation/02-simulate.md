# C39.2 — Physics::Simulate

> **The one-sentence version:** `Physics::Simulate` (`0x6BB4D0`) is the per-body tick — a `__thiscall` that gates
> ("should this body simulate?"), pre-sims the parts (`0x6A7290`), updates the contact list (`0x6A7110`), and
> ends by calling `IntegrateMotion` (`0x6BA510`).

[← C39.1 — The pipeline](01-pipeline.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md) ·
[Next: C39.3 — The part & wheel array →](03-part-array.md)

---

## The per-body tick

`Physics::Simulate` at `0x6BB4D0` is the function that ticks **one** physics body for one frame. It's reached
from the step wrapper (`0x6BB5C0`, [C39.1](01-pipeline.md)) once per body, and its job is to take the body from
its current state to its next state: check whether it should simulate, let its parts pre-compute, update its
contacts, and integrate.

Its prologue is verified:

```asm
0x6BB4D0  Physics::Simulate:
    56              push  esi
    8B F1           mov   esi, ecx          ; esi = this (the body) — __thiscall
    8B 06           mov   eax, [esi]        ; eax = vtable
    57              push  edi
    FF 50 ??        call  [eax+??]          ; virtual dispatch on the body
    ...
```

The `mov esi, ecx` is the `__thiscall` signature — the body pointer arrives in `ECX`
([C34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)). The immediate `mov eax,[esi]; call [eax+??]` is a **virtual
dispatch on the body itself** — `Simulate` calls a virtual on `this` right away, which is how a base `Simulate`
invokes the derived body's behaviour (the `RBVehicle`'s vs a generic body's,
[Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)).

> ✅ *Verified:* `Physics::Simulate` at `0x6BB4D0` begins `56 8B F1 8B 06 57 FF 50` — `push esi; mov esi,ecx;
> mov eax,[esi]; push edi; call [eax+…]`. This is a `__thiscall` that immediately dispatches a virtual on the
> body.

## The four phases

Inside `Simulate`, the body's frame is four phases:

**1. Gate — "should this body simulate?"** Early virtuals (`vtbl[+0x4C]`/`[+0x50]`) test whether the body is
active this frame — a sleeping/disabled body ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md))
skips the rest. This is the cheap early-out that keeps parked/distant cars from consuming sim time.

**2. Part pre-sim (`0x6A7290`).** The body iterates its **part array at `[this+0xEC]`**
([C39.3](03-part-array.md)) — each part (wheel, component) does its per-frame computation (suspension/tyre
forces for wheels, [Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)),
accumulating forces on the body.

**3. Contact update (`0x6A7110`).** The body's **contact list** (the collisions it's involved in,
[Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) is updated — resolving contact forces so the
integrate step accounts for collisions.

**4. Integrate (`0x6BA510`).** Finally `IntegrateMotion` ([C39.4](04-integrate.md)) applies the accumulated
forces to advance the body's motion and write the new transform.

So `Simulate` is the orchestrator: *gate → pre-sim parts → update contacts → integrate*. Each phase feeds the
next — the parts accumulate forces, contacts add collision forces, and the integrate consumes them all.

## Virtual dispatch = polymorphic bodies

The immediate virtual dispatch (`call [eax+??]`) is significant: it means `Simulate` is **polymorphic**. The
base tick calls virtuals on the body, so different body types
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md) — `RigidBody`, `SimpleRigidBody`, `RBVehicle`,
`RBCop`) plug their own behaviour into the same pipeline. A vehicle body simulates with wheels and drivetrain; a
simple prop body simulates as a plain rigid body; both go through `Physics::Simulate`, differing by their
virtuals. This is the class system ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) doing
its job — one pipeline, many body types.

## What Simulate leaves out

`Simulate` is the *physics* tick — it deliberately does **not** do input or AI. By the time `Simulate` runs, the
driver's decisions (throttle, brake, steer) are already set on the body (by input/AI earlier in the frame,
[Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)); `Simulate` reads them via the AI/INPUT mechanic
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) as inputs to the physics. This separation — decide
first, simulate second — is why the sim is deterministic given its inputs ([C39.5](05-connectors.md)): it doesn't
reach out to decide anything, it just simulates what it was told.

## RE implications

- **`Physics::Simulate (0x6BB4D0)`** is the per-body tick — `__thiscall` (`mov esi,ecx`), immediate virtual
  dispatch on the body.
- **Four phases:** gate (`vtbl[+0x4C]/[+0x50]`) → part pre-sim (`0x6A7290`) → contact update (`0x6A7110`) →
  integrate (`0x6BA510`).
- **The virtual dispatch makes it polymorphic** — one pipeline, many body types
  ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)).
- **It's the physics tick, not the decision tick** — inputs are already set; `Simulate` just simulates them.

---

### Key takeaways

- `Physics::Simulate (0x6BB4D0)` is the per-body tick — a `__thiscall` (`56 8B F1 8B 06 57 FF 50`) that
  immediately dispatches a virtual on the body.
- Its four phases are **gate → part pre-sim (`0x6A7290`) → contact update (`0x6A7110`) → integrate (`0x6BA510`)**.
- The immediate virtual dispatch makes it **polymorphic** — the same pipeline runs vehicles, cops, and simple
  bodies via their virtuals.
- `Simulate` is the **physics** tick — the driver's decisions are already set; it simulates them, it doesn't make
  them.
- Each phase feeds the next: parts and contacts accumulate forces, and `IntegrateMotion` consumes them.

**Continue:** [C39.3 — The part & wheel array](03-part-array.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md)
