# C41.2 — Physics_Base & Its Interfaces

> **The one-sentence version:** every rigid body descends from `Physics_Base`, whose constructor at `0x6B9920`
> (`6A FF 68 12 DF 87 00` — an SEH prologue) embeds **three interface sub-objects**, so one body can be held as a
> simulate-able, collide-able, and mechanic-bearing thing at once.

[← C41.1 — The RigidBody tree](01-rigidbody-tree.md) · [Chapter 41 hub](C41-Physics-RigidBody.md) ·
[Next: C41.3 — The hash unification →](03-hash-unification.md)

---

## The base of the tree

Under the whole rigid-body tree ([C41.1](01-rigidbody-tree.md)) is **`Physics_Base`** — the class that provides
the machinery every physics body shares. Its constructor is at **`0x6B9920`**, verified to open with an SEH
prologue:

```asm
0x6B9920  Physics_Base::ctor:
    6A FF              push  -1                ; SEH: end-of-chain
    68 12 DF 87 00     push  0x87DF12          ; SEH: exception handler / scope
    64 A1 00 ...       mov   eax, fs:[0]       ; SEH: install frame
    ...
```

The `push -1; push <handler>; mov eax,fs:[0]` is the standard MSVC structured-exception-handling frame setup —
`Physics_Base::ctor` installs an SEH scope because it constructs sub-objects that could throw (and must be
cleaned up correctly). That the *base* physics class does this tells you it's a real constructor doing real
sub-object construction, not a trivial init.

> ✅ *Verified:* `Physics_Base::ctor` at `0x6B9920` begins `6A FF 68 12 DF 87 00` — an SEH prologue
> (`push -1; push 0x87DF12`). The physics body is built with proper constructor/exception semantics.

## Three embedded interfaces

The defining feature of `Physics_Base` is that its constructor **embeds three interface sub-objects** — the body
*is* three interfaces at once. A physics body needs to be seen differently by different systems:

- **A simulate interface** — the sim pipeline ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
  holds the body as "a thing that ticks," calling `Simulate` on it.
- **A collision interface** — the collision world ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md))
  holds the body as "a thing that collides," querying its shape and applying contacts.
- **A third interface** — for the mechanics/attachments ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md))
  or the spatial/query system — "a thing that has a transform and parts."

Each interface is a sub-object with its own vtable ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)),
embedded at a known offset in the body. A system that wants "a collide-able" is handed the collision sub-object's
address; casting back to the full body is a fixed offset subtraction. This is C++ **multiple inheritance of
interfaces** — the body inherits several abstract interfaces and implements them all.

> 🟡 *Reasoned:* the specific identity of the three interfaces (simulate / collision / spatial) is inferred from
> how the body is used by the sim, collision, and mechanic systems, consistent with the verified three-sub-object
> constructor; the exact interface vtable layouts are deeper RE. That the ctor embeds **three** interface
> sub-objects is verified.

## Why interfaces, not one fat vtable

Building the body as several interfaces (rather than one giant class with one huge vtable) is a deliberate design:

- **Decoupling.** The collision world only knows the collision interface; it can't touch the sim state. The sim
  only knows the simulate interface. Each system depends on a narrow contract, not the whole body — the same
  isolation the connectors provide ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)).
- **Polymorphism per aspect.** Each interface can be overridden independently — a body can customise how it
  collides without touching how it simulates. The derived classes ([C41.1](01-rigidbody-tree.md)) override the
  interfaces they need to specialise.
- **Uniform handling.** Any body (`SimpleRigidBody`, `RBVehicle`, `RBCop`) presents the same interfaces, so the
  sim and collision systems handle them uniformly, regardless of the concrete type.

So `Physics_Base` is the linchpin of the physics system's architecture: it makes a body a *multi-faceted* object
that plugs into several systems through narrow, independent interfaces. This is what lets one body be simulated,
collided, and driven all at once without those systems entangling.

## The interface offset in Simulate

The interface design shows up directly in `Physics::Simulate` ([C41.4](04-simulate-thiscall.md)): the immediate
`call [eax+0x4C]` is a virtual dispatch through the body's vtable, and the `+0x4C` offset lands in one of the
interface method slots. The body's memory layout — the base fields, then the embedded interface sub-objects with
their vtables — is what makes that dispatch reach the right interface method. So the byte-level detail of the sim
([C41.4](04-simulate-thiscall.md)) is a direct consequence of the `Physics_Base` interface layout: the vtable
offsets in the sim code are indices into the interfaces this constructor builds.

## RE implications

- **`Physics_Base` is the base** of the rigid-body tree; ctor `0x6B9920` (`6A FF 68 12 DF 87 00`, SEH prologue)
  builds it.
- **It embeds three interface sub-objects** — the body is simulate-able, collide-able, and spatial at once.
- **Interfaces decouple** the sim, collision, and mechanic systems — each depends on a narrow contract.
- **The sim's `call [eax+0x4C]`** ([C41.4](04-simulate-thiscall.md)) dispatches through these interface vtables.

---

### Key takeaways

- Every rigid body descends from **`Physics_Base`** — ctor `0x6B9920`, verified SEH prologue
  (`6A FF 68 12 DF 87 00`).
- The constructor **embeds three interface sub-objects** — the body is a **simulate / collision / spatial**
  interface all at once.
- This is **C++ multiple inheritance of interfaces** — each a sub-object with its own vtable at a fixed offset.
- Interfaces **decouple** the sim, collision, and mechanic systems, letting one body plug into all of them
  independently.
- The sim's byte-level `call [eax+0x4C]` dispatch is a **direct consequence** of this interface layout.

**Continue:** [C41.3 — The hash unification](03-hash-unification.md) · [Chapter 41 hub](C41-Physics-RigidBody.md)
