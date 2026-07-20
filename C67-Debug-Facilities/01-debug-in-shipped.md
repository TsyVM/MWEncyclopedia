# C67.1 — Debug Tooling in the Shipped Exe

> **The one-sentence version:** the retail `speed.exe` carries development tooling — debug prints, assertions,
> overlays, dev screens — compiled in and shipped rather than stripped, a window into how the game was built and a
> boon to reverse-engineering.

[← Chapter 67 hub](C67-Debug-Facilities.md) · [Next: C67.2 — Debug output & assertions →](02-debug-output.md)

---

## The fossils of development

The shipped retail executable ([C58.1](../C58-Build-Pipeline/01-shipping-exe.md)) is not a stripped, minimal
binary — it's *full of development tooling*, the fossils of how Most Wanted was made:

- **Debug output** ([C67.2](02-debug-output.md)) — `DebugPrint`, `ScreenPrintf`, `DebugScreenMessage`.
- **Assertions** ([C67.2](02-debug-output.md)) — `Assert`, `Assertion`.
- **Debug overlays** ([C67.3](03-debug-overlays.md)) — `OL_CollisionDetection`.
- **Debug cameras/screens** ([C67.4](04-debug-cameras-screens.md)) — `DebugWatchCar`, `DebugVehicleSelection`.

These are the tools the *developers* used — to log state, catch bugs, visualize the physics, and jump around the
game while building it. They *shipped* in the retail binary, present (if mostly dormant) in the code the player
runs. Reading them is reading the *development environment* embedded in the game.

> ✅ *Verified:* the debug facilities are present in `speed.exe` — `DebugPrint`/`ScreenPrintf`/`DebugScreenMessage`,
> `Assert`/`Assertion`, `OL_CollisionDetection`/`OL_CollisionDetectionWidget`, `DebugWatchCar`/`DebugWorldCameraMover`,
> `DebugVehicleSelection`/`DebugCarCustomize` (12 `Debug*` names, 61 `OL_*` entries).

## Why debug tooling ships

That the debug facilities *shipped* (rather than being compiled out) is normal for a 2005 title, for several
reasons:

- **Compiled-in, conditionally-dead.** Debug code is often guarded by flags or left in with its *strings* present
  even if the code paths are disabled — the compiler keeps the strings ([C50.2](../C50-Verification-Methodology/02-byte-verification.md))
  and often the functions. Stripping them fully would require a separate build config that studios didn't always
  bother with for the retail exe.
- **Late-cut, not removed.** Debug screens (`DebugCarCustomize`, [C67.4](04-debug-cameras-screens.md)) and dev
  cameras are often *disabled* (unreachable via normal play) but *present* in code — cut from the UI flow, not the
  binary.
- **The retail build is close to the dev build.** A 2005 PC game's retail exe ([C58.1](../C58-Build-Pipeline/01-shipping-exe.md))
  is often the dev build with debug *display* turned off but the *machinery* intact — so the tooling ships as a
  byproduct.

So the debug tooling in `speed.exe` is the *development build showing through* the retail one — the instrumentation
the game was built with, dormant but present. This is common and *useful*: it means the shipped binary carries a
record of its own development, legible to RE ([below](#a-boon-to-re)).

> 🟡 *Reasoned:* that the debug facilities ship as compiled-in-but-dormant (disabled display, present code) is the
> standard outcome for a 2005 retail build, consistent with the verified debug strings and the era's build
> practices; the exact enablement state of each is per-facility RE. The debug strings and facilities are verified.

## A boon to RE

The shipped debug tooling is a *gift* to reverse-engineering ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)):

- **The developers named their tools.** `OL_CollisionDetection`, `DebugWatchCar`, `DebugVehicleSelection` — the
  debug strings *name* the systems they instrument, so a debug overlay for collision *tells you* there's a collision
  system ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)) and hints at its structure.
- **Debug prints reveal internals.** A `DebugPrint` of some value ([C67.2](02-debug-output.md)) names the value and
  its context — a window into what the code tracks.
- **Assertions state invariants.** An `Assert` ([C67.2](02-debug-output.md)) encodes a *fact the developers knew
  must hold* — a documented invariant, in the binary.

So the debug tooling is *self-documentation* by the developers — they named and instrumented their own systems, and
that instrumentation *shipped*, legible to anyone reading the strings ([C50.3](../C50-Verification-Methodology/03-hash-verification.md)).
Much of this book's ease of verification ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md))
comes from this: the game is *labelled* — by its class names, its event names, and its debug facilities. The debug
tooling is the developers' labelling of their *process*, and it's as legible as their labelling of their *systems*.
Reading it ([C67.5](05-reading-debug.md)) is reading the development, preserved in the shipped game.

## RE implications

- **The retail exe carries development tooling** — debug prints, assertions, overlays, dev screens — shipped, not
  stripped.
- **Debug tooling ships** because it's compiled-in-but-dormant (disabled display, present code) — the dev build
  showing through.
- **A boon to RE** — the developers *named* their tools, so the debug strings document the systems they instrument.
- **Self-documentation of the process** — the tooling labels the *development*, as the class names label the
  *systems*.

---

### Key takeaways

- The retail `speed.exe` is **full of development tooling** — debug prints, assertions, overlays, dev screens — the
  **fossils** of how the game was built, **shipped rather than stripped**.
- Debug tooling ships because it's **compiled-in but dormant** — disabled *display*, present *code* — the dev build
  showing through the retail one (normal for 2005).
- It's a **boon to RE** — the developers **named their own tools** (`OL_CollisionDetection`, `DebugWatchCar`), so
  the debug strings **document the systems** they instrument.
- Debug prints reveal *what the code tracks*; assertions encode *invariants the developers knew must hold* — both
  legible in the binary.
- The debug tooling is the developers' **self-documentation of their process** — as legible as their labelling of
  the systems, and much of why this book verifies so easily.

**Continue:** [C67.2 — Debug output & assertions](02-debug-output.md) · [Chapter 67 hub](C67-Debug-Facilities.md)
