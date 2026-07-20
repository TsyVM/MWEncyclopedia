# C34.1 — What a VTable Is

> **The one-sentence version:** a vtable is an array of function pointers — one per virtual method — and an
> object's first field points at it, so a virtual call reads the vtable pointer, indexes to the method's slot,
> and jumps.

[← Chapter 34 hub](C34-VTable-Anatomy.md) · [Next: C34.2 — Classifying vtable slots →](02-classifying-slots.md)

---

## The array behind every object

A **vtable** (virtual method table) is the C++ mechanism for polymorphism: a per-class array of **function
pointers**, one slot per virtual method the class declares. Every object of a class begins with a pointer to
that class's vtable ([C32.5](../C32-Runtime-Class-System/05-object-model.md)):

```
object                          vtable (the class's, at a fixed address)
+0x00  vtable pointer  ───────▶ +0x00  &method0   (e.g. constructor/destructor helper)
+0x04  field                    +0x04  &method1   (e.g. Update)
+0x08  field                    +0x08  &method2   (e.g. Render)
…      fields                    …      &methodN
```

So the vtable is the class's **behaviour**, laid out as a table of addresses, shared by all instances (the
vtable is per *class*, the fields are per *object*).

## A virtual call

Calling a virtual method is an indirect jump through the vtable:

```asm
mov  eax, [ecx]          ; ecx = this; eax = vtable pointer (object +0x00)
call [eax + slot*4]      ; call the method in that slot
```

Read the object's vtable pointer, add the method's slot offset, and call the function there. Because the slot
index is fixed per method but the vtable differs per class, the *same* call site invokes different code for
different classes — that's polymorphism. For RE, this pattern (`mov eax,[ecx]; call [eax+N]`) is the signature
of a virtual call, and `N/4` is the slot number.

## Why the vtable is where behaviour lives

The vtable is the natural place to read a class's behaviour because it **enumerates** the class's methods:

- **One table, all the virtuals.** Every overridable behaviour has a slot — the vtable is the class's interface.
- **Fixed address per class.** The vtable lives at a constant address in `speed.exe`, so it's findable and its
  slots are readable.
- **Shared by instances.** Reading one class's vtable tells you the behaviour of every object of that class.

So "read the vtable" is "read what this class can do" — the entry point to a class's behaviour
([C34.6](06-reading-behaviour.md)).

## Finding a vtable

You reach a class's vtable two ways ([C34.3](03-identify-by-vtable.md)):

- **From the constructor.** A class's constructor writes the vtable pointer into the object (`mov [object], &vtable`),
  so the constructor ([C33.3](../C33-Class-Registry-Factories/03-construction.md)) names the vtable.
- **From an object.** A live object's `+0x00` *is* the vtable pointer — read it to find the class's vtable.

Once you have the vtable address, you read its slots as an array of function pointers until the array ends (the
next vtable or non-code data begins) — giving the method count ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).

> ✅ *Verified:* the vtable-pointer-first object layout and virtual-call pattern are the standard C++ ABI
> confirmed in `speed.exe` ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)); vtable method
> counts were measured per class ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).

## RE implications

- **The vtable pointer is object `+0x00`** — read it to find the class's vtable
  ([C34.3](03-identify-by-vtable.md)).
- **A virtual call is `mov eax,[this]; call [eax+N]`** — `N/4` is the slot.
- **The vtable enumerates the class's virtuals** — read its slots to read its behaviour
  ([C34.6](06-reading-behaviour.md)).
- **Reach a vtable from the constructor or from an object** — both point at the same table.

---

### Key takeaways

- A **vtable** is a per-class array of method pointers; an object's `+0x00` points at it.
- A virtual call reads the vtable pointer, indexes the method's slot, and jumps — the polymorphism mechanism.
- The vtable **enumerates a class's behaviour** at a fixed address — the entry point to reading it.
- Find a vtable from a constructor (writes it) or from an object (`+0x00` is it).
- The call pattern `mov eax,[this]; call [eax+N]` marks a virtual call; `N/4` is the slot.

**Continue:** [C34.2 — Classifying vtable slots](02-classifying-slots.md) · [Chapter 34 hub](C34-VTable-Anatomy.md)
