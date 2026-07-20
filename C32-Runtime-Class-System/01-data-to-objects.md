# C32.1 — Data Becomes Live Objects

> **The one-sentence version:** a file on disk is inert; the game instantiates it as a class **object** — a
> block of memory with a vtable and fields — that holds state and updates every frame, so the class system is
> the bridge from data to behaviour.

[← Chapter 32 hub](C32-Runtime-Class-System.md) · [Next: C32.2 — The five class roles →](02-five-roles.md)

---

## From inert data to live behaviour

Everything in Parts I–VI was **data at rest** — a car's tuning is numbers in a vault
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), a cop is a reference, a trigger is a polygon
([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)). None of it *does* anything until the game
**instantiates** it as an **object**: a live C++ class instance that occupies memory, holds state, and runs
code each frame ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)).

```
data on disk (vault, geometry, save)
   → load → construct an object of the right class
   → the object holds state + runs behaviour each frame
```

So the class system is the **bridge**: it turns "a car's numbers" into "a driving vehicle object," "a cop
reference" into an `AIVehicleCopCar`, "a trigger polygon" into a live trigger that fires messages. Behaviour
lives in objects; data configures them.

## An object is memory + code

A live object is two things bound together:

- **State** — a block of memory (the object's **fields**) holding its current values: position, speed, health,
  timers, references to other objects ([C32.5](05-object-model.md)).
- **Behaviour** — the class's methods, reached through the object's **vtable**
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)), that read and update that state.

Data configures the state (a car's tuning sets its performance fields); the class supplies the behaviour (the
vehicle simulation that uses them). This is why the same class (a vehicle) produces different cars — same
behaviour, different data-driven fields ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).

## Configuration vs identity

A crucial distinction the class system draws:

- **The class** is the *identity* — what kind of thing this is (a cop car, a suspension component, an AI goal).
  It fixes the behaviour, size, and vtable.
- **The data** is the *configuration* — the specific values that make this instance distinct (this cop's
  aggression, this car's tuning).

So a fully-tuned M3 GTR and a base sedan can be the *same class* (a player vehicle) with *different data*.
Understanding a runtime object means identifying its class (behaviour) and reading its data (state) — the class
tells you what it does, the data tells you how it's set up.

## Why a class system at all

Structuring the runtime as registered classes ([C32.4](04-registration.md)) rather than ad-hoc code buys the
engine:

- **Uniformity.** Every entity, component, and system is an object with a vtable, updated the same way — the
  frame loop ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) walks objects, not special
  cases.
- **Data-driven construction.** The right class is instantiated for the right data
  ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)), so new content is new data,
  not new code.
- **Reflection.** Classes register by name/hash ([C32.4](04-registration.md)), so the runtime can look up and
  construct classes generically.

This is the substrate everything else in the running game sits on — the reason the formats and the behaviour
connect.

## For the reverse-engineer

Crossing into the runtime changes the tools ([C32.6](06-reading-binary.md)): instead of parsing file bytes, you
read the **executable** — classes, vtables, and object layouts in `speed.exe`. The payoff is understanding not
just *what the data is* but *what the game does with it* — the behaviour the data configures. The class system
is the map from one to the other.

---

### Key takeaways

- Data on disk is inert; the game **instantiates** it as class **objects** that hold state and run each frame.
- The class system is the **bridge** from data (configuration) to behaviour (objects).
- An object is **state** (fields) + **behaviour** (vtable methods); data sets the fields, the class sets the
  behaviour.
- Class = **identity** (what it is), data = **configuration** (how it's set up) — same class, different data =
  different instances.
- A class system gives uniform updates, data-driven construction, and reflection — the runtime substrate; RE it
  in `speed.exe`.

**Continue:** [C32.2 — The five class roles](02-five-roles.md) · [Chapter 32 hub](C32-Runtime-Class-System.md)
