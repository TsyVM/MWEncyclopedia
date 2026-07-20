# C34.5 — Inheritance in VTables

> **The one-sentence version:** a derived class's vtable begins with its base's slots (same methods at the same
> indices) and appends its own, so `AIVehicleCopCar`'s 324 slots are the accumulated hierarchy — base vehicle,
> AI vehicle, cop car — read top to bottom as base to most-derived.

[← C34.4 — Method roles & the common interface](04-method-roles.md) · [Chapter 34 hub](C34-VTable-Anatomy.md) ·
[Next: C34.6 — Reading behaviour from a vtable →](06-reading-behaviour.md)

---

## Derived vtables extend base ones

C++ single inheritance lays out a derived vtable as **base slots first, derived slots appended**:

```
Base vtable:        slot0..slotN         (base's virtuals)
Derived vtable:     slot0..slotN         (base's virtuals — overridden or inherited)
                    slotN+1..slotM       (derived's new virtuals)
```

So the derived class's vtable **starts identical in shape** to the base's — same methods at the same slot
indices — with two differences:

- **Overrides** — a slot the derived class overrides points at the derived implementation (same slot, different
  function).
- **Inherited** — a slot the derived class doesn't override still points at the base implementation (same slot,
  same function).
- **Additions** — new virtuals the derived class declares get appended slots.

This is why the common interface ([C34.4](04-method-roles.md)) is at the *top* of every vtable in a hierarchy —
it's the base's slots, shared by all descendants.

## Reading a vtable is reading the hierarchy

Because slots accumulate down the inheritance chain, reading a vtable top-to-bottom is reading the class
hierarchy **base → most-derived**:

```
AIVehicleCopCar vtable (324 slots)
├── [base object]     slots for the common base (destructor, update, …)
├── [vehicle base]    vehicle slots (physics, render, drive)
├── [AI vehicle]      AI slots (goals, actions — Ch 46)
└── [cop car]         cop-specific slots (pursuit, formation, bust — Ch 49)
```

Each band of slots corresponds to a level of the hierarchy. So `AIVehicleCopCar`'s 324 slots aren't 324
independent behaviours — they're the *sum* of a base vehicle's, an AI vehicle's, and a cop car's virtuals. This
is why it's the largest vtable ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)):
it inherits from the most.

## Identifying overrides

The interesting slots in a derived vtable are the **overrides** — where the derived class replaces a base
method with its own. You spot them by comparing the derived vtable to its base's
([C34.3](03-identify-by-vtable.md)):

- **Same function as base** → inherited (the base behaviour, unchanged).
- **Different function from base** → overridden (the derived class's specialised behaviour) — read these.
- **Beyond the base's length** → a new derived method.

So diffing a class's vtable against its base's isolates exactly what the derived class *changes* — its
contribution to the hierarchy. This is the most efficient way to understand a derived class: read only its
overrides and additions, inheriting the rest from the base you already understand.

> 🟡 *Reasoned:* the base-then-derived vtable layout and override/inherited distinction are the standard C++
> single-inheritance ABI, consistent with the verified object model and the large accumulated vtables
> ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)); the exact per-class band
> boundaries are per-hierarchy RE.

## Multiple inheritance and thunks

Where a class uses **multiple inheritance**, extra vtables and **thunks** ([C34.2](02-classifying-slots.md))
appear — adjustor stubs that fix the `this` pointer for a secondary base before jumping to the real method
(`add ecx, offset; jmp method`). So a thunk-heavy region of a vtable signals multiple inheritance
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)'s connectors and mechanics may show this).
Recognising thunks keeps you from mistaking an adjustor for a real method.

## RE implications

- **Read base → derived** — a vtable's slots accumulate down the hierarchy.
- **Diff against the base** to find **overrides** — the derived class's actual changes.
- **Inherit the rest** — slots identical to the base are the base's behaviour, already understood.
- **Thunks signal multiple inheritance** — adjustors, not real methods ([C34.2](02-classifying-slots.md)).

---

### Key takeaways

- A derived vtable is **base slots first, derived slots appended** — same methods at the same indices, plus
  overrides and additions.
- Reading a vtable top-to-bottom reads the **hierarchy base → most-derived** (bands per level).
- `AIVehicleCopCar`'s 324 slots are the **accumulated** base vehicle + AI vehicle + cop car virtuals.
- **Diff against the base** to isolate overrides (the class's real changes); inherit the rest.
- **Thunks** mark multiple inheritance — adjustors, not behaviour.

**Continue:** [C34.6 — Reading behaviour from a vtable](06-reading-behaviour.md) · [Chapter 34 hub](C34-VTable-Anatomy.md)
