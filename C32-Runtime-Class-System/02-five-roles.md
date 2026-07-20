# C32.2 — The Five Class Roles

> **The one-sentence version:** runtime classes play five roles — entities, mechanic components, AI, managers/
> systems, and connectors — and knowing a class's role is most of understanding what it does before you read a
> single method.

[← C32.1 — Data becomes live objects](01-data-to-objects.md) · [Chapter 32 hub](C32-Runtime-Class-System.md) ·
[Next: C32.3 — The eleven families →](03-eleven-families.md)

---

## Roles group the classes by purpose

Beneath the eleven registration families ([C32.3](03-eleven-families.md)) lie a handful of **roles** — the kinds
of purpose a class serves in the running game. Classifying a class by role is the fastest way to understand it:

- **Entities** — the *things* in the world: vehicles (player, cop, traffic), world bodies, props. They have a
  position, a physics presence, and are what you see and hit.
- **Mechanic components** — the *parts* that make an entity work: engine, transmission, suspension, tyres, etc.
  (the vehicle mechanics, [Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)). They compute the
  entity's behaviour.
- **AI** — the *minds*: the goals and actions ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md))
  that drive non-player entities — what a cop or racer decides to do.
- **Managers & systems** — the *coordinators*: the named systems and managers that own populations and
  orchestrate gameplay (traffic, pursuit, the session).
- **Connectors** — the *wiring*: objects that link the others, carrying data across boundaries (e.g. between AI
  and vehicle, [Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).

Five roles cover everything the runtime instantiates: things, their parts, their minds, their coordinators, and
the wires between.

## Role → behaviour

The role tells you a class's *shape of behaviour* before you decode it:

- An **entity** has a transform, a body, and an update that moves it — look for physics and rendering.
- A **mechanic** takes inputs and produces forces/values — look for a `Simulate`/compute method
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)).
- An **AI** class has goals/actions and a decision update — look for a planner
  ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)).
- A **manager** owns a list and coordinates it — look for population/spawn logic.
- A **connector** moves data one way ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) — look
  for a thin pass-through.

So when you meet an unfamiliar class in `speed.exe` ([C32.6](06-reading-binary.md)), placing it in a role
narrows what its methods must be doing.

## Roles and families

The five roles map onto the eleven families ([C32.3](03-eleven-families.md)) — families are the *registration
lists*, roles are the *conceptual groupings*:

| Role | Families |
|---|---|
| Entities | world bodies, players, (AI vehicles) |
| Mechanic components | vehicle mechanics (51 — the largest) |
| AI | AI goals, AI actions, director actions |
| Managers & systems | managers & activities, named systems, world tasks |
| Connectors | connectors, devices |

The families are finer (a registration detail); the roles are coarser (a comprehension tool). Both partition the
same class population — [C32.3](03-eleven-families.md) gives the exact families and counts.

> 🟡 *Reasoned:* the five roles are the archive's comprehension grouping over the ✅ verified eleven families
> ([C32.3](03-eleven-families.md)); the families, their list-heads, and counts are verified in `speed.exe`.

## Why role-first understanding works

Leading with role is efficient because a role fixes most of a class's story:

- **Role + size** ([C32.5](05-object-model.md)) is often enough — a 1964-byte entity with 324 methods
  (`AIVehicleCopCar`) is obviously a heavyweight actor; a tiny connector is obviously a wire.
- **Role predicts the vtable shape** ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) — an entity's
  vtable has update/render slots; a mechanic's has a simulate slot.
- **Role guides the search** — knowing you're looking at a "manager" tells you to find its owned list.

So classifying by role first, then confirming with size/vtable, is the method for reading the class system —
formalised in [C32.6](06-reading-binary.md).

---

### Key takeaways

- Runtime classes play five **roles**: entities, mechanic components, AI, managers/systems, connectors.
- The role fixes a class's *shape of behaviour* — what its methods must broadly do — before you decode them.
- Roles map onto the eleven registration families (finer detail); both partition the same class population.
- **Mechanic components** are the largest role (51 families entries) — the vehicle sim dominates.
- Lead with role, confirm with size/vtable — the efficient way to read an unfamiliar class.

**Continue:** [C32.3 — The eleven families](03-eleven-families.md) · [Chapter 32 hub](C32-Runtime-Class-System.md)
