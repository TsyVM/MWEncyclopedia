# Chapter 67 — Debug & Development Facilities

> **Goal of this chapter:** decode the developer tooling left in the *shipped* retail executable — the debug output
> (`DebugPrint`, `ScreenPrintf`, `DebugScreenMessage`), the assertion system (`Assert`/`Assertion`), the debug
> overlays (`OL_CollisionDetection`/`Widget`), and the debug cameras/screens (`DebugWatchCar`,
> `CDActionDebug`, `DebugVehicleSelection`) — the development scaffolding visible in `speed.exe`.

A shipped game carries the *fossils* of its development — the debug print statements, assertions, overlays, and dev
shortcuts the developers used, left in the retail binary. This chapter decodes those facilities in `speed.exe`:
what they are, why they shipped, and what they reveal about how Most Wanted was *built and debugged*. It's the
development-side companion to the build chapter ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)) — where
that decoded the *toolchain*, this decodes the *debugging tools* baked into the game.

> **Verified against the executable.** The dev facilities are named in `speed.exe`: **debug output** —
> `DebugPrint`, `ScreenPrintf`, `DebugScreenMessage`, `DebugStringA`, `DebugBreak`; the **assertion system** —
> `Assert`, `Assertion`, `asserts`; **debug overlays** — `OL_CollisionDetection`, `OL_CollisionDetectionWidget`
> (among 61 `OL_*` overlay/online screens); **debug cameras/screens** — `DebugWatchCar`, `DebugWorldCameraMover`,
> `DebugWorld`, `DebugCarCustomize`/`DebugCarCustomizeScreen`, `DebugVehicleSelection`, `DebugCarOption`; and the
> **debug director actions** — `CDActionDebug`, `CDActionDebugWatchCar`
> ([C53.3](../C53-Cameras-Director/03-cinematic-director.md)). `console`/`ConsoleCtrlHandler` and `ProfileManager`
> are present.

---

## Deep-dive pages

- [C67.1 — Debug tooling in the shipped exe](01-debug-in-shipped.md): the fossils of development, and why they
  ship.
- [C67.2 — Debug output & assertions](02-debug-output.md): `DebugPrint`/`ScreenPrintf` and `Assert`.
- [C67.3 — Debug overlays](03-debug-overlays.md): `OL_CollisionDetection` and visualizing internal state.
- [C67.4 — Debug cameras & screens](04-debug-cameras-screens.md): `DebugWatchCar`, dev shortcuts.
- [C67.5 — Reading debug facilities in RE](05-reading-debug.md): what the tooling reveals.

---

## 67.1 Debug tooling in the shipped exe

The retail `speed.exe` ([C58.1](../C58-Build-Pipeline/01-shipping-exe.md)) is full of **development tooling**
([C67.1](01-debug-in-shipped.md)) — debug prints, assertions, overlays, dev screens. These *shipped* (rather than
being stripped) because they're compiled-in facilities that were left enabled or left as dead-but-present code — a
normal outcome for a 2005 title ([C58.1](../C58-Build-Pipeline/01-shipping-exe.md)). They're a *window* into
development: the tools the developers built to make, tune, and debug the game.

## 67.2 Debug output & assertions

The most basic facilities are **debug output** and **assertions** ([C67.2](02-debug-output.md)): `DebugPrint`/
`DebugStringA` (log to the debugger), `ScreenPrintf`/`DebugScreenMessage` (print *on screen*, via the `ScreenPrintf.fng`
overlay, [Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)), and `Assert`/`Assertion` (check invariants, break on
violation). These are the developers' *eyes* — how they saw the game's internal state and caught bugs. That they're
present shows the codebase was *instrumented* throughout ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).

## 67.3 Debug overlays

The **debug overlays** ([C67.3](03-debug-overlays.md)) — `OL_CollisionDetection`/`OL_CollisionDetectionWidget` —
*visualize* internal state on screen: the collision-detection overlay draws the collision world
([Chapter 63](../C63-Collision-World/C63-Collision-World.md)) — the AABBs, the contacts — so developers could *see*
the physics ([C43.1](../C43-Collision-Contacts/01-detection.md)) that's normally invisible. Built on the same FEng
`Widget` system ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) as the HUD, the overlays are dev-facing
visualizations of the engine's hidden data.

## 67.4 Debug cameras & screens

The **debug cameras and screens** ([C67.4](04-debug-cameras-screens.md)) are dev *shortcuts*: `DebugWatchCar`/
`CDActionDebugWatchCar` ([C53.3](../C53-Cameras-Director/03-cinematic-director.md)) — a camera to *watch* a specific
car (a cop, a racer) for debugging its behaviour ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md));
`DebugWorldCameraMover`/`DebugWorld` — a free-fly debug camera; `DebugVehicleSelection`/`DebugCarCustomize` —
dev screens to jump to any car or customization without the normal flow. These let developers *inspect and
manipulate* the game directly, bypassing the player-facing systems.

---

### Key takeaways

- The retail `speed.exe` carries **development tooling** — debug prints, assertions, overlays, dev screens — the
  *fossils* of how the game was built and debugged, shipped rather than stripped.
- **Debug output** (`DebugPrint`/`ScreenPrintf`/`DebugScreenMessage`) and **assertions** (`Assert`/`Assertion`)
  were the developers' *eyes* — logging state and catching invariant violations.
- **Debug overlays** (`OL_CollisionDetection`/`Widget`) *visualize* hidden state — drawing the collision world
  ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)) so developers could *see* the physics.
- **Debug cameras/screens** (`DebugWatchCar`, `DebugWorldCameraMover`, `DebugVehicleSelection`) are dev
  **shortcuts** — inspect a car's AI, free-fly the world, jump to any vehicle.
- These facilities are a **window into development** — and a boon to RE, because the developers *named* their own
  tools ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).

**Next:** *This completes the C59–C67 systems batch.* See [C67.5](05-reading-debug.md) for how the debug tooling
grounds the whole book's method.
