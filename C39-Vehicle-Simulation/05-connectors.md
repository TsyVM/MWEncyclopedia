# C39.5 — The One-Way Connector Boundary

> **The one-sentence version:** the vehicle sim is isolated from the rest of the game by **connectors** — the
> 8-class connector family (`0x00988EC0`) — that pass data in **one direction**: inputs flow *into* the sim,
> results flow *out*, so the physics never reaches into the systems that read it.

[← C39.4 — IntegrateMotion](04-integrate.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md) ·
[Next: C39.6 — Input to tyres, end to end →](06-input-to-tyres.md)

---

## The connector family

The sim is wired to the game through **connectors** — a distinct class family. The runtime class system has
**eleven family list-heads** ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)), and one of them, at
**`0x00988EC0`**, is the connector family with **8 registered classes**. A connector is an object whose job is to
*carry data across a boundary* — from a producer to a consumer — without the two sides knowing about each other.

In the vehicle sim, connectors are what tie the physics body to the things around it: the input/AI that drives
it, the render object that draws it, the audio that sounds it, the game logic that scores it. Each of those links
is a connector, and the set of them is the sim's interface to the rest of the engine.

> ✅ *Verified:* the connector class family is one of the eleven family list-heads
> ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)), at `0x00988EC0`, with 8 registered classes.

## One-way by design

The crucial property is that the boundary is **one-way**. Data flows through a connector in a single direction:

- **Into the sim:** the driver's input/AI decisions ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md))
  — throttle, brake, steer — flow *in* through a connector, set on the body before `Simulate` runs
  ([C39.2](02-simulate.md)).
- **Out of the sim:** the body's results — position (transform `[this+0xF0]`), speed, orientation
  ([C39.4](04-integrate.md)) — flow *out* through connectors to the renderer, audio, HUD, AI, and game logic.

A given connector carries data one way. The sim doesn't reach out to *pull* input (input is pushed in); the
consumers don't reach in to *pull* results (results are pushed out, or read from a stable published field). This
directionality is what keeps the sim a clean, isolated unit.

## Why one-way matters: determinism and isolation

The one-way boundary buys two things:

**Determinism.** Because the sim only reads inputs that were set *before* it runs, and only writes outputs, its
result is a pure function of its inputs and its previous state. It doesn't depend on the order it's ticked
relative to consumers, or on any consumer's state. Given the same inputs, the same body simulates the same way —
which is what makes the physics reproducible and debuggable ([C39.7](07-reading-sim.md)).

**Isolation.** The physics is decoupled from everything that reads it. The renderer, audio, and HUD read the
sim's published outputs; they can't perturb the sim. This means the sim can be reasoned about (and reverse-engineered)
as a self-contained system: forces in, motion out. It also means the sim can run on its own schedule (e.g.,
fixed-step) independent of the variable-rate consumers.

So the connector boundary is not just plumbing — it's the architectural choice that makes the vehicle physics a
predictable, isolated core that the rest of the game observes but does not disturb.

> 🟡 *Reasoned:* the one-way data-flow semantics (inputs pushed in before `Simulate`, outputs published for
> consumers) are the standard connector/component decoupling, consistent with the verified 8-class connector
> family and the sim's structure-anchored isolation ([C39.1](01-pipeline.md)); the per-connector field layout is
> deeper RE.

## Connectors vs. the object model

Connectors are objects like any other ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) —
constructed via the factory ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)),
with vtables ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)), interned into the connector family list
at `0x00988EC0`. That connectors are their own *family* (not lumped with the bodies or the AI) reflects their
distinct role: they're the wiring, a first-class part of the object model, separate from the things they wire.
When you enumerate the connector family in a dump ([C32.6](../C32-Runtime-Class-System/06-reading-binary.md)),
you're seeing the game's data-flow graph — every producer→consumer link in the running simulation.

## RE implications

- **The sim is wired via connectors** — the 8-class family at `0x00988EC0`
  ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)).
- **The boundary is one-way** — inputs in (before `Simulate`), results out (after `IntegrateMotion`).
- **One-way = determinism + isolation** — the sim is a pure function of its inputs, decoupled from consumers.
- **Connectors are a first-class family** — the game's data-flow graph, enumerable in a dump.

---

### Key takeaways

- The vehicle sim is wired to the game through **connectors** — one of the eleven class families, at
  `0x00988EC0`, with 8 classes.
- The boundary is **one-way**: driver input/AI flows *in* before `Simulate`; position/speed/orientation flow
  *out* after `IntegrateMotion`.
- One-way data flow gives **determinism** (the sim is a pure function of its inputs) and **isolation** (consumers
  observe but don't disturb).
- Connectors are **first-class objects** in their own family — the running game's producer→consumer graph.
- This isolation is why the vehicle physics can be reverse-engineered as a self-contained "forces in, motion out"
  core.

**Continue:** [C39.6 — Input to tyres, end to end](06-input-to-tyres.md) · [Chapter 39 hub](C39-Vehicle-Simulation.md)
