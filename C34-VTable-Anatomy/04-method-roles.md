# C34.4 — Method Roles & the Common Interface

> **The one-sentence version:** classes in a family share a base interface — the same virtual methods at the
> same early slots (constructor/destructor, update/tick, role-specific methods) — so a vtable's first slots are
> predictable by role, and its later slots are the class's specifics.

[← C34.3 — Identifying a class from its vtable](03-identify-by-vtable.md) · [Chapter 34 hub](C34-VTable-Anatomy.md) ·
[Next: C34.5 — Inheritance in vtables →](05-inheritance.md)

---

## A shared interface at the top

Classes don't have arbitrary vtables — they inherit a **base interface** ([C34.5](05-inheritance.md)), so their
vtables begin with the same methods at the same slots. The early slots are the **common interface** every class
of a family ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)) implements:

- **Destructor** — teardown ([C33.3](../C33-Class-Registry-Factories/03-construction.md)), at a conventional
  slot.
- **Update / Tick** — the per-frame method the frame loop calls
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) — the heartbeat of a live object.
- **Role-specific base methods** — a mechanic's `Simulate`
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)), an entity's render/collide, an AI's decision
  method ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)).

So knowing a class's role ([C32.2](../C32-Runtime-Class-System/02-five-roles.md)) predicts its early slots — you
recognise the shared interface at a glance and focus on what's different.

## The update method is the heartbeat

The most important shared method is the **update/tick** — the one the frame loop calls each frame
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) to advance the object. Every live object
has one, and reading it is reading what the object *does per frame*:

- An **entity's** update integrates motion, handles input/AI, updates state.
- A **mechanic's** update (`Simulate`) computes its contribution to the vehicle
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).
- An **AI's** update runs its planner ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)).

So finding the update slot and reading its method is the single most productive vtable read — it's where the
object's frame behaviour lives ([C34.6](06-reading-behaviour.md)).

## Role predicts the interface

Each role ([C32.2](../C32-Runtime-Class-System/02-five-roles.md)) has a characteristic interface shape:

| Role | Characteristic methods |
|---|---|
| Entity | update, render, collide, damage |
| Mechanic | `Simulate` (compute contribution) ([Ch 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) |
| AI goal/action | evaluate/select, execute ([Ch 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) |
| Manager | update population, spawn/despawn |
| Connector | read/write pass-through ([Ch 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) |

So classifying a class's role ([C34.3](03-identify-by-vtable.md)) tells you which methods to look for in its
vtable — a targeted read rather than a blind scan.

> 🟡 *Reasoned:* the common-interface slots (destructor, update, role methods) are the standard OO framework
> shape, consistent with the verified object model and family structure
> ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)); the exact slot indices are per-family
> RE.

## Later slots are the specifics

After the shared interface, a vtable's **later slots** are the class's own methods — the specialised behaviour
that distinguishes it from its siblings ([C34.5](05-inheritance.md)):

- `AIVehicleCopCar`'s later slots are cop-specific (pursuit maneuvers, formation, bust logic —
  [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).
- A specific mechanic's later slots are its particular computation.

So a vtable reads as **shared interface (top) + specifics (bottom)**: recognise the top by role, read the bottom
for what makes this class distinct. This structure is what makes even a 324-slot vtable
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)) navigable.

## RE implications

- **Recognise the common interface** at early slots by role — destructor, update, role methods
  ([C32.2](../C32-Runtime-Class-System/02-five-roles.md)).
- **Read the update method first** — it's the object's per-frame behaviour
  ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)).
- **Role predicts the interface** — look for the methods a role implies.
- **Later slots are specifics** — read them for what distinguishes the class from its family.

---

### Key takeaways

- Classes share a **common interface** at early vtable slots (destructor, update/tick, role-specific base
  methods).
- The **update/tick** method is the object's per-frame heartbeat — the most productive slot to read.
- **Role predicts the interface** — each role (entity, mechanic, AI, manager, connector) has characteristic
  methods.
- **Later slots are the class's specifics** — a vtable is shared interface (top) + specifics (bottom).
- Recognise the top by role, read the update first, then the specific later slots.

**Continue:** [C34.5 — Inheritance in vtables](05-inheritance.md) · [Chapter 34 hub](C34-VTable-Anatomy.md)
