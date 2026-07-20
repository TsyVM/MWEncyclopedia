# C32.5 — The Object Model

> **The one-sentence version:** a live object is a fixed-size block of memory beginning with a vtable pointer
> and followed by its fields — so a class's identity is its size and vtable, and `AIVehicleCopCar` (1964 bytes,
> 324 methods) is the game's most behaviour-packed object.

[← C32.4 — Registration: name → hash → list-head](04-registration.md) · [Chapter 32 hub](C32-Runtime-Class-System.md) ·
[Next: C32.6 — Reading the class system from the binary →](06-reading-binary.md)

---

## What a live object is

A runtime object is a C++ instance — a block of memory laid out as:

```
object (size bytes)
+0x00  vtable pointer   → the class's virtual method table (Chapter 34)
+0x04  field
+0x08  field
…      fields (the object's state)
+size  (end)
```

Three facts define the object model:

- **A fixed size** — every instance of a class is the same number of bytes (`AIVehicleCopCar` = 1964).
- **A vtable pointer first** — the object begins with a pointer to its class's virtual method table
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)), the standard C++ layout.
- **Fields after** — the rest is the object's state (position, speed, references, timers), configured by data
  ([C32.1](01-data-to-objects.md)).

So an object is **size + vtable + fields**: how big it is, what it can do (vtable), and its current state
(fields).

## Size and method count fingerprint a class

A class's **size** and **vtable method count** are its fingerprint — enough to identify it and gauge its
weight:

- **`AIVehicleCopCar`** — 1964 bytes, 324 vtable methods. The largest registered class: a cop car is an entity
  *and* an AI *and* a physics body, packed with behaviour.
- **A small connector** — a few dozen bytes, a handful of methods: a thin wire ([C32.2](02-five-roles.md)).

The size measures state (how much an object remembers); the method count measures behaviour (how much it can
do). Together they place a class on the spectrum from "heavyweight actor" to "lightweight wire" before you read
any code ([C32.6](06-reading-binary.md)).

> ✅ *Verified (archive Discovery 10):* measured object sizes and vtable method counts — `AIVehicleCopCar` is
> 1964 bytes / 324 methods, the largest; the object model (vtable pointer + fields) is the standard C++ layout
> confirmed in the disassembly.
> 🟡 *Reasoned:* the exact field layout within a given class is per-class RE; the size/vtable-count fingerprints
> are verified.

## Inheritance and shared layout

Classes inherit, and the object model reflects it: a derived class's object begins with its base class's layout
(same fields at the same offsets) and adds its own fields after, with a vtable that extends the base's
([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)). So:

- A vehicle base class defines the common fields (transform, body); a cop car adds cop-specific state.
- The shared prefix means base-class code works on any derived object (polymorphism via the vtable).

This is why the families ([C32.3](03-eleven-families.md)) and roles ([C32.2](02-five-roles.md)) work — classes
in a family share a base layout and vtable shape, so the runtime processes them uniformly.

## The vtable is the behaviour

The vtable pointer at `+0x00` is the object's link to its **behaviour** — the array of method pointers the class
implements ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)). Calling a virtual method looks it up in
the vtable and jumps, so:

- **The object's data** (fields) is what it *is*.
- **The object's vtable** is what it *does*.
- **Identifying a class** from an object is reading its vtable pointer and matching it to a known class
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).

So the object model ties the two halves of Chapter 32.1 together: state in the fields, behaviour through the
vtable, bound in one memory block.

## RE implications

- **Identify a class by its vtable pointer** ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) — the
  value at object `+0x00`.
- **Gauge a class by size + method count** — the fingerprint ([C32.6](06-reading-binary.md)).
- **Read fields at offsets** — object state is at fixed field offsets after the vtable pointer.
- **Respect inheritance** — a derived object contains its base's layout as a prefix.

---

### Key takeaways

- A live object is **size + vtable + fields**: a fixed-size block, a vtable pointer at `+0x00`, then state.
- **Size** (state) and **vtable method count** (behaviour) fingerprint a class — `AIVehicleCopCar` = 1964 B /
  324 methods, the largest.
- Inheritance gives a shared base-layout prefix and an extended vtable — the basis of polymorphism and uniform
  processing.
- The vtable pointer links the object to its **behaviour**; the fields hold what it *is*.
- Identify classes by vtable pointer, gauge them by size/method-count, read state at field offsets, respect
  inheritance.

**Continue:** [C32.6 — Reading the class system from the binary](06-reading-binary.md) · [Chapter 32 hub](C32-Runtime-Class-System.md)
