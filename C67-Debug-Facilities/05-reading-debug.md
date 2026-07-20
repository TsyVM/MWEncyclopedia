# C67.5 — Reading Debug Facilities in RE

> **The one-sentence version:** the debug tooling is the developers' self-documentation — every `Debug*` name,
> `Assert`, and `OL_*` overlay *labels the system it instruments* — so reading the debug facilities is reading the
> game's own account of how it was built, and the ground of the whole book's verification method.

[← C67.4 — Debug cameras & screens](04-debug-cameras-screens.md) · [Chapter 67 hub](C67-Debug-Facilities.md) ·
[Book index →](../README.md)

---

## Anchors for debug-facility RE

The debug facilities are anchored on verified strings ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)):

- **Debug output** — `DebugPrint`, `DebugStringA`, `ScreenPrintf`, `DebugScreenMessage` ([C67.2](02-debug-output.md)).
- **Assertions** — `Assert`, `Assertion`, `asserts`, `DebugBreak` ([C67.2](02-debug-output.md)).
- **Debug overlays** — `OL_CollisionDetection`, `OL_CollisionDetectionWidget` ([C67.3](03-debug-overlays.md)).
- **Debug cameras** — `DebugWatchCar`, `CDActionDebugWatchCar`, `DebugWorldCameraMover`, `DebugWorld`
  ([C67.4](04-debug-cameras-screens.md)).
- **Debug screens** — `DebugVehicleSelection`, `DebugCarCustomize`/`Screen`, `DebugCarOption`
  ([C67.4](04-debug-cameras-screens.md)).

From these, the whole development-tooling layer is navigable: the output, the checks, the visualizations, and the
dev interfaces.

## The RE workflow

Reading the debug facilities:

1. **Find the output** — `DebugPrint`/`ScreenPrintf` ([C67.2](02-debug-output.md)); what the developers logged.
2. **Read the assertions** — `Assert` ([C67.2](02-debug-output.md)); the documented invariants.
3. **Trace the overlays** — `OL_CollisionDetection` ([C67.3](03-debug-overlays.md)); the visualized systems.
4. **Follow the dev interfaces** — `DebugWatchCar`/`DebugVehicleSelection` ([C67.4](04-debug-cameras-screens.md));
   how the developers navigated the game.

The output is a picture of *how the game was debugged* — and, through the systems the tooling names, a map of the
game itself ([below](#debug-tooling-as-a-map)).

## Debug tooling as a map

The deepest use of the debug facilities in RE is as a *map of the whole engine* ([C67.1](01-debug-in-shipped.md)) —
because each debug facility *names and touches the system it instruments*:

- **`OL_CollisionDetection`** → the collision system ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)).
- **`DebugWatchCar`** → the AI ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) and pursuit
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) systems it watches.
- **`DebugVehicleSelection`/`DebugCarCustomize`** → the vehicle
  ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) and customization
  ([Chapter 56](../C56-Customization/C56-Customization.md)) systems.
- **`DebugWorld`** → the world ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) and streaming
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) systems.

So the debug facilities form a *directory* of the engine's systems — each `Debug*`/`OL_*` name points at a system,
and the debug *code* (in RE) is a *read path into that system* ([C67.3](03-debug-overlays.md)). To debug collision,
the developers wrote code that reads and draws the collision world — so `OL_CollisionDetection`'s code is a guided
tour of collision ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)). The debug tooling is thus a
*meta-map*: not a system itself, but a set of *pointers into* every system, each with example code for reading it.
For RE, this is invaluable — the debug facilities are the developers' *own index* of the engine.

## Companion to the build chapter

This chapter is the *runtime* companion to the build chapter ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md))
— together they decode *how Most Wanted was made*:

- **The build pipeline** ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)) — the *toolchain*: the
  compiler ([C58.1](../C58-Build-Pipeline/01-shipping-exe.md)), the linker, the asset pipeline
  ([C58.4](../C58-Build-Pipeline/04-asset-pipeline.md)) — how the game was *built*.
- **The debug facilities** (this chapter) — the *debugging tools*: the output, assertions, overlays, dev screens —
  how the game was *debugged and tuned*.

So Chapter 58 decoded the *construction* (compiling and packaging) and this chapter the *debugging* (observing and
fixing) — the two halves of *development*. A game is *built* ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md))
and *debugged* (this chapter), and *both* leave fossils in the shipped binary
([C58.1](../C58-Build-Pipeline/01-shipping-exe.md)): the build leaves toolchain signatures, the debugging leaves the
`Debug*` facilities. Reading both reconstructs the *development environment* — the tools and process behind the
game — from the game itself. This is the "how it was made" thread of the book, and the debug facilities are its most
direct evidence: the developers' actual tools, shipped.

## The ground of the book's method

Finally, the debug facilities are the *ground* of the whole book's verification method
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) — they exemplify *why* the game is
so readable:

- **The game is labelled.** Every system has a name — class names ([C50.3](../C50-Verification-Methodology/03-hash-verification.md)),
  event names, and *debug names* — and those names *shipped* ([C67.1](01-debug-in-shipped.md)). The book's method
  ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) rests on this labelling.
- **The developers documented their own code.** Assertions ([C67.2](02-debug-output.md)) state invariants, overlays
  ([C67.3](03-debug-overlays.md)) name systems, debug screens ([C67.4](04-debug-cameras-screens.md)) name flows —
  *self-documentation* baked in.
- **The verification method reads that documentation.** Every ✅ in this book
  ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)) is a string, a byte pattern, a hash the
  developers *left in the binary* — and the debug facilities are the densest such documentation.

So the debug facilities are the *reason* the book can be verification-first
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)): the game *documents itself*, in its
names, its assertions, and its debug tooling, and that documentation *shipped* in `speed.exe`. The whole
book is, in a sense, an act of *reading the developers' own labelling* — and the debug facilities are where that
labelling is most explicit, most abundant, and most directly about *the game's own systems*. Reading the debug
tooling is reading the game's account of itself: how it was built ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)),
how it was debugged (this chapter), and — through the systems it names — *what it is* ([the whole book](../README.md)).
It is fitting that the systems batch ends here, at the tools the developers used to *understand their own game* —
the same task, from the same evidence, this book has undertaken throughout.

## RE implications

- **Anchor on** the `Debug*` strings, `Assert`, and `OL_CollisionDetection` — the verified debug facilities.
- **The RE workflow** — output → assertions → overlays → dev interfaces.
- **Debug tooling as a map** — each facility names and reads a system; a meta-map of the engine.
- **Companion to Chapter 58** — build (construction) + debug (this chapter) = how the game was made.
- **The ground of the book's method** — the game documents itself; the debug facilities are the densest such
  documentation.

---

### Key takeaways

- The debug facilities are anchored on verified strings — **`DebugPrint`/`ScreenPrintf`** (output), **`Assert`**
  (invariants), **`OL_CollisionDetection`** (overlays), **`DebugWatchCar`/`DebugVehicleSelection`** (dev interfaces).
- The RE workflow: **output → assertions → overlays → dev interfaces** — a picture of *how the game was debugged*.
- The debug tooling is a **map of the engine** — each `Debug*`/`OL_*` name **points at a system**, and the debug
  code is a **read path into it**; a meta-map with example code for reading every system.
- This chapter is the **runtime companion to the build chapter** ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md))
  — build (construction) + debug (observation) = **how Most Wanted was made**, both leaving fossils in the shipped
  binary.
- The debug facilities are the **ground of the book's verification method** — the game **documents itself** in its
  names, assertions, and debug tooling, and that documentation **shipped**; reading it is reading the game's own
  account of itself.

**This completes the C59–C67 systems batch.** See the [book index](../README.md) for the full chapter map.

**Sources:** `speed.exe` (verified strings: `DebugPrint`, `DebugStringA`, `ScreenPrintf`, `DebugScreenMessage`,
`DebugBreak`; `Assert`, `Assertion`, `asserts`; `OL_CollisionDetection`, `OL_CollisionDetectionWidget` [among 61
`OL_*`]; `DebugWatchCar`, `CDActionDebugWatchCar`, `DebugWorldCameraMover`, `DebugWorld`; `DebugVehicleSelection`,
`DebugCarCustomize`, `DebugCarCustomizeScreen`, `DebugCarOption`; `console`, `ConsoleCtrlHandler`, `ProfileManager`).
Director actions ([C53.3](../C53-Cameras-Director/03-cinematic-director.md)); collision world
([Chapter 63](../C63-Collision-World/C63-Collision-World.md)); build pipeline
([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)); verification method
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)).
