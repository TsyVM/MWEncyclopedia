# C34.2 — Classifying VTable Slots

> **The one-sentence version:** a vtable's slots fall into five kinds — real method, getter, thunk, stub, and
> destructor — and classifying them turns a 324-slot wall of addresses into a short list of the real behaviours
> worth reading.

[← C34.1 — What a vtable is](01-what-is-a-vtable.md) · [Chapter 34 hub](C34-VTable-Anatomy.md) ·
[Next: C34.3 — Identifying a class from its vtable →](03-identify-by-vtable.md)

---

## Five kinds of slot

A big vtable ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) is not 324
behaviours — most slots are trivial. Classifying each slot by the function it points at focuses attention:

| Kind | What the function looks like | Meaning |
|---|---|---|
| **Real method** | non-trivial code (branches, calls, state) | genuine behaviour — the slots that matter |
| **Getter** | `mov eax, [ecx+N]; ret` or `mov eax, imm; ret` | returns a field or constant |
| **Thunk** | `add ecx, N; jmp method` | adjusts `this`, jumps to the real method (inheritance) |
| **Stub** | `ret` or `xor eax,eax; ret` | empty/unimplemented optional virtual |
| **Destructor** | frees/tears down, at a conventional slot | the class's teardown |

Classifying a vtable is walking its slots and bucketing each function. The **real methods** are the small
fraction worth deep-reading; the rest are recognised and skipped.

## Recognising each kind

Each kind has a byte signature you can spot quickly ([C4](../C4-Byte-Level-Toolcraft/C4-Byte-Level-Toolcraft.md)):

- **Getter** — two or three instructions ending in `ret`, returning a field (`mov eax,[ecx+N]`) or a constant
  (`mov eax,imm`). The allocator impostor ([C35.1](../C35-Memory-Management/C35-Memory-Management.md)) is exactly
  this shape (`mov eax,0x9205E0; ret`) — a getter mistaken for more.
- **Thunk** — an adjustor: `add ecx, offset; jmp realMethod`. It fixes the `this` pointer for a sub-object and
  tail-jumps; no real work of its own.
- **Stub** — a bare `ret` (or `xor eax,eax; ret`): the class declares the virtual but does nothing (an optional
  hook it doesn't use).
- **Destructor** — recognisable by freeing the object / running field destructors, at the class's
  destructor slot ([C33.3](../C33-Class-Registry-Factories/03-construction.md)).
- **Real method** — everything else: branches, loops, calls, meaningful state changes.

> ✅ *Verified (archive Discovery 10):* the slot-classification taxonomy (method/getter/thunk/stub/destructor)
> is the method used to read the game's vtables; the getter-stub pattern is confirmed (e.g. the allocator
> impostor `0x6269B0` = `mov eax,0x9205E0; ret`, [C35.1](../C35-Memory-Management/C35-Memory-Management.md)).

## Why so many trivial slots

A vtable with hundreds of slots but few real methods is normal, for structural reasons:

- **Inheritance accumulates slots.** A deep hierarchy ([C34.5](05-inheritance.md)) inherits every base's
  virtuals; a leaf class's vtable includes all of them, most unchanged (so pointing at base implementations —
  effectively "inherited" slots).
- **Optional hooks.** Frameworks declare many overridable hooks; a class implements a few and stubs the rest.
- **Getters as virtuals.** Accessors are often virtual, filling slots with trivial getters.

So the ratio of real methods to slots is low, and classifying is what separates the ~dozens of behaviours from
the hundreds of slots. This is why a 324-slot class ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md))
is *understandable* — most slots are trivial or inherited.

## The payoff: a short list of real methods

After classification, a vtable reduces to a short list of **real methods** — the behaviours to read
([C34.6](06-reading-behaviour.md)):

```
vtable (324 slots)
  → classify: ~40 real methods, ~150 getters, ~80 inherited/thunks, ~50 stubs, 1 destructor  (illustrative)
  → read the ~40 real methods (Update, Simulate, collision handlers, …)
```

The real methods, guided by the class's role ([C32.2](../C32-Runtime-Class-System/02-five-roles.md)) and common
interface ([C34.4](04-method-roles.md)), are the class's actual behaviour. Classification is the filter that
makes a huge vtable tractable.

## RE implications

- **Classify before deep-reading** — bucket each slot; only the real methods need real analysis.
- **Recognise trivial slots by signature** — getters, thunks, stubs are two-or-three-instruction patterns.
- **Watch the getter-stub trap** — a trivial getter can be mistaken for a real function
  ([C35.1](../C35-Memory-Management/C35-Memory-Management.md)); classify by behaviour.
- **Most slots are inherited/trivial** — the real behaviour is a small fraction; find it and focus.

---

### Key takeaways

- VTable slots are five kinds: **real method, getter, thunk, stub, destructor** — classify each to find the real
  behaviour.
- Each kind has a byte signature (getter: `mov eax,…;ret`; thunk: `add ecx,N;jmp`; stub: `ret`) — spot them fast.
- A big vtable has many trivial/inherited slots — inheritance, optional hooks, and virtual getters fill it.
- Classification reduces a 324-slot vtable to a **short list of real methods** — the tractable behaviour.
- Classify before deep-reading; recognise trivial slots by signature; beware the getter-stub trap.

**Continue:** [C34.3 — Identifying a class from its vtable](03-identify-by-vtable.md) · [Chapter 34 hub](C34-VTable-Anatomy.md)
