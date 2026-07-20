# C34.6 — Reading Behaviour from a VTable

> **The one-sentence version:** to understand a class, classify its vtable slots, read the update method and the
> overrides first, and follow their calls — turning a table of addresses into a description of what the class
> does each frame.

[← C34.5 — Inheritance in vtables](05-inheritance.md) · [Chapter 34 hub](C34-VTable-Anatomy.md) ·
[Next: Chapter 35 — Memory Management & Allocation →](../C35-Memory-Management/C35-Memory-Management.md)

---

## The reading procedure

Turning a vtable into an understanding of a class is a focused procedure that composes the chapter:

1. **Classify the slots** ([C34.2](02-classifying-slots.md)) — bucket into real methods / getters / thunks /
   stubs / destructor; keep the real methods.
2. **Recognise the common interface** ([C34.4](04-method-roles.md)) — by role, identify the destructor, update,
   and role base methods at the early slots.
3. **Diff against the base** ([C34.5](05-inheritance.md)) — isolate the **overrides** and **additions** (the
   class's real changes).
4. **Read the update method first** ([C34.4](04-method-roles.md)) — the per-frame behaviour, the class's
   heartbeat.
5. **Read the overrides/additions** — the specialised behaviour distinguishing this class.
6. **Follow their calls** — into fields ([C32.5](../C32-Runtime-Class-System/05-object-model.md)) and other
   objects/systems.

Steps 1–3 cheaply reduce the vtable to a short list; steps 4–6 read the behaviour that matters.

## Start with update, then overrides

The two highest-value reads are the **update method** and the **overrides**:

- **Update** ([C34.4](04-method-roles.md)) — what the object does every frame
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)). For an entity, this is its whole
  behaviour loop; for a mechanic, its `Simulate`; for an AI, its planner tick.
- **Overrides** ([C34.5](05-inheritance.md)) — what *this* class changes from its base. If a cop car overrides
  the vehicle's update, the override is the cop-specific behaviour on top of the vehicle behaviour.

Reading these two answers "what does this class do, and what makes it special" — most of the class's story.

## Fields give the state, methods give the behaviour

Behaviour and state are read together ([C32.5](../C32-Runtime-Class-System/05-object-model.md)):

- **The methods** (vtable) tell you *what the class does*.
- **The fields** (object layout) tell you *what it operates on* — the state the methods read and write.

So as you read a method, note the fields it touches (`[ecx+N]`), building the object's field map alongside its
behaviour. The vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) often names those fields
(same reflection hash, [C32.4](../C32-Runtime-Class-System/04-registration.md)), so a field an override reads may
be a tuning value you already decoded — data and code meeting again.

## Worked shape: a vehicle class

Reading a vehicle class's vtable ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) illustrates
the payoff:

```
classify → find the update slot (the per-frame vehicle sim)
read update → it calls each mechanic's Simulate (Ch 40), integrates motion (Ch 41), handles collision (Ch 43)
overrides → a cop car adds pursuit behaviour (Ch 49); a racer adds AI (Ch 47)
fields → speed, RPM, position — configured by the vault (Ch 13)
```

So the vtable read reveals the vehicle pipeline ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
from the inside — the update method *is* the per-frame simulation, and the overrides are the per-variant
behaviour. The later simulation and AI chapters are, in effect, guided reads of these vtables.

## The skill that unlocks the runtime

VTable reading is the core runtime-RE skill because it's how you go from *structure* to *behaviour*:

- The registry ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) gives you the
  classes.
- The object model ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) gives you their
  layout.
- The **vtable** (this chapter) gives you their **behaviour**.

With it, you can read what any class does, which is what the rest of Parts VIII–IX
([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
does for the simulation and AI — read the vtables of the vehicle, physics, and AI classes to reconstruct the
game's behaviour.

## RE implications

- **Follow the six-step read** — classify → recognise interface → diff base → read update → read overrides →
  follow calls.
- **Update + overrides first** — the highest-value behaviour.
- **Map fields alongside methods** — state and behaviour together; cross-reference the vault
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
- **VTable reading unlocks the runtime** — it's how structure becomes behaviour.

---

### Key takeaways

- Reading a class: **classify slots → recognise the interface → diff the base → read update → read overrides →
  follow calls**.
- The **update method** and the **overrides** are the highest-value reads — per-frame behaviour and per-class
  specifics.
- Read **fields alongside methods** — state (fields) + behaviour (methods); the vault often names the fields.
- A vehicle class's vtable *is* the per-frame simulation ([Ch 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md));
  the later chapters are guided vtable reads.
- VTable reading is the skill that turns class structure into **behaviour** — the key to the runtime.

**Continue:** [Chapter 35 — Memory Management & Allocation](../C35-Memory-Management/C35-Memory-Management.md) ·
[Chapter 34 hub](C34-VTable-Anatomy.md)
