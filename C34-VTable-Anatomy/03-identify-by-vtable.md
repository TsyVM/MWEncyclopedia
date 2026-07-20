# C34.3 — Identifying a Class from Its VTable

> **The one-sentence version:** each class has a unique vtable at a fixed address, so an object's vtable pointer
> identifies its class — read the pointer at object `+0x00`, look it up in the catalogue, and the anonymous
> object has a name.

[← C34.2 — Classifying vtable slots](02-classifying-slots.md) · [Chapter 34 hub](C34-VTable-Anatomy.md) ·
[Next: C34.4 — Method roles & the common interface →](04-method-roles.md)

---

## A vtable pointer is a class ID

Every class's vtable lives at a **unique, fixed address** in `speed.exe`. So an object's vtable pointer
([C32.5](../C32-Runtime-Class-System/05-object-model.md)) — the value at `+0x00` — is effectively a **class
identifier**: two objects with the same vtable pointer are the same class; different pointers, different
classes. This makes vtable-pointer matching the most reliable class-identification method
([C33.5](../C33-Class-Registry-Factories/05-fingerprints.md)):

```python
def identify(obj_addr, memory, vtable_catalogue):
    vptr = read_u32(memory, obj_addr + 0x00)     # the object's vtable pointer
    return vtable_catalogue.get(vptr)            # → the class (name, family, size)
```

Size ([C33.5](../C33-Class-Registry-Factories/05-fingerprints.md)) narrows candidates, but classes can share a
size; the vtable pointer is *unique*, so it names the class outright.

## Building the vtable catalogue

To identify by vtable you need the **catalogue** — a map from vtable address to class
([C33.4](../C33-Class-Registry-Factories/04-class-reference.md)). You build it from the registrations
([C33.2](../C33-Class-Registry-Factories/02-factory-registration.md)):

- Each class's **constructor** writes its vtable pointer into the object (`mov [obj], &vtable`), so the
  constructor names the vtable.
- The **registration** ([C33.2](../C33-Class-Registry-Factories/02-factory-registration.md)) links name →
  constructor → vtable.
- Collecting these gives `vtable address → class name` for every registered class.

With the catalogue, any vtable pointer resolves to a class name — the runtime becomes browsable by object
([C33.6](../C33-Class-Registry-Factories/06-using-registry.md)).

## Identifying a live object

In practice, identifying an object you encounter (tracing the frame loop,
[Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md), or inspecting a data structure):

1. **Read `+0x00`** — the vtable pointer.
2. **Look it up** in the vtable catalogue → the class name.
3. **Confirm** with the object's size ([C33.5](../C33-Class-Registry-Factories/05-fingerprints.md)) — it should
   match the class's size.
4. **Now you know** its role, size, behaviour ([C34.6](06-reading-behaviour.md)), and can read its fields.

So a pointer in memory becomes a named, understood object. This is the core move of runtime RE: **from an
address to a class.**

> ✅ *Verified:* the object's vtable pointer at `+0x00` identifies its class (standard C++ layout,
> [Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)); vtable→class mapping comes from the
> registrations ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).

## Why this beats other methods

Vtable identification is more reliable than the alternatives:

- **vs size** — sizes are shared; a 1964-byte object could be several classes, but its vtable is one.
- **vs field patterns** — fields can look similar across classes; the vtable is definitive.
- **vs guessing from context** — context suggests, the vtable confirms.

So when you need to *know* what an object is, read its vtable pointer. Size and context are hints; the vtable is
the answer.

## Handling unknowns

If a vtable pointer isn't in your catalogue (an un-catalogued class):

- **Read the vtable's slots** ([C34.2](02-classifying-slots.md)) — classify them to infer the class's behaviour.
- **Match by shape** — the vtable's method count and common-interface slots ([C34.4](04-method-roles.md)) suggest
  its family/role.
- **Recover the name** — if the class registered, hash-match its key ([C33.6](../C33-Class-Registry-Factories/06-using-registry.md)).

So even an unknown vtable is analysable — its slots reveal the class's nature even before you have a name.

## RE implications

- **Vtable pointer = class ID** — the definitive identification.
- **Build the vtable→class catalogue** from registrations/constructors.
- **Identify objects** by reading `+0x00` and looking it up (confirm with size).
- **Unknown vtable?** Classify its slots to infer behaviour; hash-match to recover a name.

---

### Key takeaways

- Each class has a **unique fixed-address vtable**, so an object's vtable pointer (`+0x00`) **identifies its
  class**.
- Build a **vtable→class catalogue** from registrations/constructors, then any pointer resolves to a class.
- Identify a live object: read `+0x00` → look up → confirm with size → know its role/behaviour/fields.
- Vtable identification beats size (shared) and field patterns (ambiguous) — it's definitive.
- Unknown vtables are still analysable — classify their slots and hash-match to recover a name.

**Continue:** [C34.4 — Method roles & the common interface](04-method-roles.md) · [Chapter 34 hub](C34-VTable-Anatomy.md)
