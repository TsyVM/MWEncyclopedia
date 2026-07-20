# C41.4 — Physics::Simulate, Byte by Byte

> **The one-sentence version:** `Physics::Simulate` (`0x6BB4D0`) decodes to `push esi; mov esi,ecx; mov eax,[esi];
> push edi; call [eax+0x4C]` — a `__thiscall` that takes the body in `ECX` and immediately dispatches its virtual
> at **vtable offset `+0x4C`**, the polymorphic entry to the per-body tick.

[← C41.3 — The hash unification](03-hash-unification.md) · [Chapter 41 hub](C41-Physics-RigidBody.md) ·
[Next: C41.5 — IntegrateMotion & the math →](05-integrate-math.md)

---

## The bytes

`Physics::Simulate` at `0x6BB4D0` is verified to the byte:

```
56              push  esi
8B F1           mov   esi, ecx          ; esi = this
8B 06           mov   eax, [esi]        ; eax = this->vtable
57              push  edi
FF 50 4C        call  dword ptr [eax+0x4C]   ; virtual dispatch, slot +0x4C
```

Byte string: `56 8B F1 8B 06 57 FF 50 4C`. Every part of this is meaningful, and together they tell the whole
story of how a body ticks.

> ✅ *Verified:* `Physics::Simulate` at `0x6BB4D0` = `56 8B F1 8B 06 57 FF 50 4C` = `push esi; mov esi,ecx;
> mov eax,[esi]; push edi; call [eax+0x4C]`. It is a `__thiscall` (`this` in `ECX`) that dispatches a virtual at
> vtable offset `+0x4C`.

## `mov esi, ecx` — the `__thiscall`

The `mov esi, ecx` is the signature of a **`__thiscall`** ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)):
the `this` pointer arrives in `ECX` (the MSVC calling convention for C++ member functions), and `Simulate`
immediately saves it in `ESI` (a callee-saved register it can keep across the calls that follow). So the first
thing `Simulate` establishes is "the body I'm ticking is in `ESI`" — everything after operates on that body.

This confirms `Physics::Simulate` is a **member function of the body** — it's `body->Simulate()`, dispatched on a
rigid body ([C41.1](01-rigidbody-tree.md)). The `push esi; push edi` save the two registers `Simulate` will use
across its internal calls (the part pre-sim, contact update, integrate,
[C39.2](../C39-Vehicle-Simulation/02-simulate.md)).

## `mov eax,[esi]; call [eax+0x4C]` — the virtual dispatch

The heart of the prologue is the two instructions `mov eax,[esi]; call [eax+0x4C]`:

- **`mov eax,[esi]`** loads the body's **vtable pointer** — the first field of any object with virtuals
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) is its vtable, so `[esi]` (= `[this+0]`) is the
  vtable.
- **`call [eax+0x4C]`** calls the function pointer at **offset `+0x4C`** in that vtable — a virtual method,
  dispatched dynamically based on the body's actual type.

So `Simulate`'s *first action* is to call a virtual on the body at slot `+0x4C`. This is the **polymorphic
entry**: because the call goes through the body's own vtable, each body type ([C41.1](01-rigidbody-tree.md))
supplies its own function there. A `SimpleRigidBody` runs its `+0x4C` method, an `RBVehicle` runs its (different)
`+0x4C` method, an `RBCop` its. One `Simulate`, dispatched to many behaviours — the class tree
([C41.1](01-rigidbody-tree.md)) doing its work at the byte level.

The `+0x4C` offset (slot 19 in the vtable, at 4 bytes each) lands in the body's interface method table
([C41.2](02-physics-base.md)) — it's one of the simulate-interface's virtuals. This is the concrete link between
the `Physics_Base` interface layout ([C41.2](02-physics-base.md)) and the sim code: the offset the sim calls is
an index into the interface this body's constructor built.

## Why dispatch first?

`Simulate` dispatching a virtual *immediately* (before doing anything else) is a common orchestration pattern:
the base `Simulate` is a thin shell that hands off to the body's type-specific tick right away. The base provides
the *entry* (the function the sim driver calls, [C39.1](../C39-Vehicle-Simulation/01-pipeline.md)), and the
virtual provides the *behaviour*. This keeps the pipeline uniform — the sim driver always calls
`Physics::Simulate` — while letting each body type do its own thing, dispatched at `+0x4C`.

It also means the "four phases" of the tick ([C39.2](../C39-Vehicle-Simulation/02-simulate.md)) — gate, part
pre-sim, contact update, integrate — are orchestrated *through* this virtual, in the body's type-specific method.
The gate virtuals (`vtbl[+0x4C]`/`[+0x50]`, [C39.2](../C39-Vehicle-Simulation/02-simulate.md)) are the very slots
dispatched here: the sim asks the body "should you simulate, and how?" through its vtable, and the body answers
with its type's behaviour.

## RE implications

- **`Physics::Simulate (0x6BB4D0)` is a `__thiscall`** — `this` in `ECX` (`mov esi,ecx`), a member function of
  the body.
- **It dispatches a virtual at vtable offset `+0x4C`** immediately (`mov eax,[esi]; call [eax+0x4C]`) — the
  polymorphic entry.
- **The `+0x4C` slot is per body type** ([C41.1](01-rigidbody-tree.md)) — one `Simulate`, many behaviours; the
  offset indexes the `Physics_Base` interface ([C41.2](02-physics-base.md)).
- **The bytes confirm the architecture** — the class tree and interface layout are visible in the sim's
  9-byte prologue.

---

### Key takeaways

- `Physics::Simulate (0x6BB4D0)` = `56 8B F1 8B 06 57 FF 50 4C` = `push esi; mov esi,ecx; mov eax,[esi]; push edi;
  call [eax+0x4C]` — **verified to the byte**.
- `mov esi,ecx` marks it a **`__thiscall`** — a member function of the rigid body (`this` in `ECX`).
- `mov eax,[esi]; call [eax+0x4C]` is an **immediate virtual dispatch** at vtable slot `+0x4C` — the polymorphic
  entry to the per-body tick.
- The `+0x4C` slot is supplied **per body type** (the class tree) and indexes the **`Physics_Base` interface**
  (C41.2) — the architecture is visible in the bytes.
- The base `Simulate` is a **thin shell** that hands off to the body's type-specific behaviour right away.

**Continue:** [C41.5 — IntegrateMotion & the math](05-integrate-math.md) · [Chapter 41 hub](C41-Physics-RigidBody.md)
