# Chapter 34 — VTable Anatomy & Method Roles

> **Goal of this chapter:** read a class's behaviour from its virtual method table — how to find a vtable,
> classify its slots (real method / getter / thunk / stub / destructor), and identify a class from its vtable
> alone — the core skill of runtime reverse-engineering.

Chapter 32 established that a live object begins with a **vtable pointer**, and the vtable is where its
**behaviour** lives. This chapter is how to read it: a vtable is an array of function pointers, and learning to
classify those slots turns a wall of addresses into a legible interface — and lets you name an object from its
vtable pointer alone.

> **Grounded in verified evidence.** The object model (vtable pointer at `+0x00`, then fields) is the standard
> C++ layout confirmed in `speed.exe` ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)),
> and the class catalogue measured each class's vtable method count — `AIVehicleCopCar` has **324** vtable
> methods, the most in the game ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).
> The slot-classification method below is drawn from that vtable evidence (archive Discovery 10).

---

## Deep-dive pages

- [C34.1 — What a vtable is](01-what-is-a-vtable.md): the array of method pointers behind every object.
- [C34.2 — Classifying vtable slots](02-classifying-slots.md): method / getter / thunk / stub / destructor.
- [C34.3 — Identifying a class from its vtable](03-identify-by-vtable.md): naming an object from its vtable
  pointer.
- [C34.4 — Method roles & the common interface](04-method-roles.md): the shared slots every class implements.
- [C34.5 — Inheritance in vtables](05-inheritance.md): how derived vtables extend base ones.
- [C34.6 — Reading behaviour from a vtable](06-reading-behaviour.md): from slots to what a class does.

---

## 34.1 A vtable is an array of method pointers

A **vtable** (virtual method table) is an array of function pointers — one per virtual method the class
implements. An object's first field ([C32.5](../C32-Runtime-Class-System/05-object-model.md)) points at its
class's vtable, so calling a virtual method is: read the vtable pointer, index to the method's slot, jump
([C34.1](01-what-is-a-vtable.md)). The vtable *is* the class's behaviour, laid out as a table — which is why
reading it reveals what a class does.

## 34.2 Not all slots are equal

The slots in a vtable are not all "real methods." Classifying them is the key skill
([C34.2](02-classifying-slots.md)):

- **Real method** — a genuine function implementing behaviour (the interesting slots).
- **Getter** — a tiny function returning a field or constant (like the allocator impostor's `mov eax,X; ret`,
  [C35.1](../C35-Memory-Management/C35-Memory-Management.md)).
- **Thunk** — an adjustor that fixes `this` and jumps to the real method (multiple inheritance).
- **Stub** — an empty or `ret`-only slot (an unimplemented/optional virtual).
- **Destructor** — the class's teardown, at a conventional slot.

A 324-slot vtable ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) isn't 324
behaviours — many are getters, thunks, and stubs. Classifying them focuses attention on the real methods.

## 34.3 A vtable pointer names a class

Because each class has a **unique vtable at a fixed address**, an object's vtable pointer
([C32.5](../C32-Runtime-Class-System/05-object-model.md)) **identifies its class**
([C34.3](03-identify-by-vtable.md)). So naming an anonymous object in memory is: read its vtable pointer, look it
up in the catalogue ([C33.4](../C33-Class-Registry-Factories/04-class-reference.md)). This is the most reliable
class-identification method — more so than size, which classes can share.

## 34.4 A common interface

Classes in a family ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)) share a **base interface** — the
same virtual methods at the same early slots ([C34.4](04-method-roles.md)): a constructor/destructor, an
update/tick, and role-specific methods (a mechanic's `Simulate`, an entity's render). So the *first* slots of a
vtable are predictable by role, and the *later* slots are the class's specific behaviour. Knowing the common
interface lets you read the shared slots at a glance and focus on the distinctive ones.

## 34.5 Inheritance extends vtables

A derived class's vtable **begins with its base's slots** (same methods at the same indices) and **appends its
own** ([C34.5](05-inheritance.md)). So `AIVehicleCopCar`'s 324 slots include the base vehicle's slots, the AI
vehicle's additions, and the cop car's specifics — the accumulation of an inheritance chain. Reading a vtable
top-to-bottom is reading the class hierarchy from base to most-derived.

---

### Key takeaways

- A **vtable** is an array of method pointers; an object's vtable pointer (`+0x00`) links it to its behaviour.
- Slots are **not all real methods** — classify them: method / getter / thunk / stub / destructor.
- A vtable pointer **uniquely identifies a class** — the most reliable identification (look it up in the
  catalogue).
- Classes share a **common interface** at early slots (constructor/destructor, update, role methods); specifics
  come later.
- Inheritance **extends** vtables (base slots then derived); a 324-slot vtable is an accumulated hierarchy
  (`AIVehicleCopCar`).

**Next:** [Chapter 35 — Memory Management & Allocation](../C35-Memory-Management/C35-Memory-Management.md): where
objects are allocated.
