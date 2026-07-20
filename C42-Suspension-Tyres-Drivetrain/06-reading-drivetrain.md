# C42.6 — Reading the Drivetrain in RE

> **The one-sentence version:** navigate the drivetrain and suspension by their verified vtables (`EngineRacer`
> `0x008AB6A0`, `SuspensionRacer` `0x008ABAC0`, …), their reflection-hash vault keys, and the mechanics list-head
> `0x0092C660` — reading the driving model from code (classes) and data (vault) together.

[← C42.5 — The tuning surface](05-tuning-surface.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md) ·
[Next: Chapter 43 — Collision Detection & Contact Records →](../C43-Collision-Contacts/C43-Collision-Contacts.md)

---

## Anchors for drivetrain RE

The drivetrain/suspension model is anchored on verified structures:

- **The vtables** — `EngineRacer` (`0x008AB6A0`), `EngineTraffic` (`0x008AB8F8`), `EngineDragster` (`0x008ABF34`),
  `EngineSpline` (`0x008AB7AC`); `SuspensionRacer` (`0x008ABAC0`), `SuspensionSimple` (`0x008ABC28`),
  `SuspensionTraffic` (`0x008ABB80`), `SuspensionSpline` (`0x008ABD88`), `SuspensionTrailer` (`0x008ABCE0`) —
  each a real array of method pointers ([C42.1](01-fidelity-tiers.md)).
- **The hashes** — the reflection hashes of the class names as vault keys ([C42.5](05-tuning-surface.md)):
  `EngineRacer` `0xB2809518`, `SuspensionRacer` `0x6209E06A`, etc.
- **The list-head** — `0x0092C660`, where the mechanics register
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).
- **The `*Params` schemas** — `SuspensionParams` (and engine equivalents), the tunable field shape
  ([C42.3](03-suspension.md)).

From these, the driving model is fully navigable: the code (vtables), the data (vault keys), and the registration.

## The RE workflow

Reading the driving model:

1. **Enumerate the families** — the `Engine*`/`Suspension*` vtables ([C42.1](01-fidelity-tiers.md)); confirm each
   by counting method pointers (the counts verify the class).
2. **Find each car's specs** — hash the spec name a car references ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md))
   and read its parameters from `attributes.bin` ([C42.5](05-tuning-surface.md)).
3. **Trace the mechanic run** — the specs register on `0x0092C660` and run within `Physics::Simulate`
   ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)); the part pre-sim
   ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)) is where the per-wheel work happens.
4. **Read a car's tune** — the vault parameters ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
   are the car's gear ratios, spring rates, NOS — the full tune.

The output is the whole driving model of a car: which fidelity classes it uses and what numbers configure them.

## Verifying a class

The cleanest verification in this chapter is the **method-count check** — a strong test that a claimed vtable is
real:

```python
# a vtable is a run of pointers into .text; count them
def count_vtable(va, exe, IB=0x400000):
    fo = va - IB; n = 0
    while True:
        p = int.from_bytes(exe[fo+n*4:fo+n*4+4], 'little')
        if 0x401000 <= p < IB+len(exe):   # points into code?
            n += 1
        else:
            break
    return n
# count_vtable(0x008AB6A0) == 123  → EngineRacer verified
```

Every class in this chapter was verified this way: the vtable address holds a run of exactly *N* code pointers,
and *N* matches the stated method count (EngineRacer 123, SuspensionRacer 45, SuspensionTraffic 86, …). A vtable
that ends where non-code data begins gives an unambiguous count — the class's *size in behaviour*. This is the
verification-first discipline ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md))
applied to classes: a claimed class is real if its vtable is a clean run of method pointers of the stated length.

> ✅ *Verified:* all nine `Engine*`/`Suspension*` vtables in this chapter were confirmed by the method-count check
> — each holds exactly the stated number of consecutive `.text` pointers (`EngineRacer` 123, `EngineTraffic` 67,
> `EngineDragster` 132, `EngineSpline` 56, `SuspensionRacer` 45, `SuspensionSimple` 44, `SuspensionTraffic` 86,
> `SuspensionSpline` 57, `SuspensionTrailer` 99).

## The driving model in one view

Pulling the chapter together, a car's driving model is:

```
car's vault collection (Chapter 13)
   ├─ engine spec  (EngineRacer 0xB2809518)  → torque, gears, NOS   → wheel torque
   ├─ suspension   (SuspensionRacer 0x6209E06A) → springs, ride, roll → wheel loads
   └─ tyre params  → grip, slip response
        ↓ (per-wheel, in the part pre-sim, C39.3)
   longitudinal + lateral tyre forces (friction circle, C42.4)
        ↓
   rigid body integrates (Chapter 41) → the car moves
```

Code (the verified classes) reads data (the vault), computes forces per wheel, and the rigid body integrates them.
That's the whole driving model — and every box in it is anchored on a verified vtable, hash, or offset. This is
why the vehicle chapters ([39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)–42) are among the most
grounded in the book: the driving model is visible in code and data at once.

## RE implications

- **Anchor on the vtables, hashes, list-head, and `*Params` schemas** — the driving model's fixed points.
- **The RE workflow** — enumerate families → find each car's specs → trace the run → read the tune.
- **Verify a class by its method count** — a clean run of `.text` pointers of the stated length (all nine
  confirmed).
- **The driving model is code + data** — verified classes reading vault parameters, per wheel, into the rigid
  body.

---

### Key takeaways

- The drivetrain/suspension model is anchored on **verified vtables** (`EngineRacer` `0x008AB6A0`, …), **vault-key
  hashes**, the **list-head `0x0092C660`**, and the **`*Params` schemas**.
- The RE workflow: **enumerate the families → find each car's specs (by hash) → trace the mechanic run → read the
  tune**.
- The strongest check is the **method count** — a real vtable is a clean run of `.text` pointers of the stated
  length; all nine classes were confirmed this way.
- The full driving model is **code reading data**: verified `Engine*`/`Suspension*` classes consuming vault
  parameters, computing per-wheel tyre forces, integrated by the rigid body.
- The vehicle chapters (39–42) are **among the most grounded** in the book — the driving model is visible in code
  and data at once.

**Next:** [Chapter 43 — Collision Detection & Contact Records](../C43-Collision-Contacts/C43-Collision-Contacts.md):
how the bodies touch the world.

**Sources:** `speed.exe` (verified: `Engine*`/`Suspension*` vtables and method counts — `EngineRacer` `0x008AB6A0`
/123, `EngineTraffic` `0x008AB8F8`/67, `EngineDragster` `0x008ABF34`/132, `EngineSpline` `0x008AB7AC`/56,
`SuspensionRacer` `0x008ABAC0`/45, `SuspensionSimple` `0x008ABC28`/44, `SuspensionTraffic` `0x008ABB80`/86,
`SuspensionSpline` `0x008ABD88`/57, `SuspensionTrailer` `0x008ABCE0`/99; tyre states `ETireBlown`/`ETirePunctured`);
`GLOBAL/attributes.bin` (verified: spec reflection hashes as vault keys — `EngineRacer` ×4, `EngineTraffic` ×2,
`SuspensionRacer` ×3, `SuspensionTraffic` ×2, `SuspensionSimple` ×1, `SuspensionTrailer` ×1).
