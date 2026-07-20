# C67.2 — Debug Output & Assertions

> **The one-sentence version:** the debug I/O is `DebugPrint`/`DebugStringA` (log to the debugger), `ScreenPrintf`/
> `DebugScreenMessage` (print on screen), and `Assert`/`Assertion` (check invariants and break on violation) — the
> developers' eyes and safety net.

[← C67.1 — Debug tooling in the shipped exe](01-debug-in-shipped.md) · [Chapter 67 hub](C67-Debug-Facilities.md) ·
[Next: C67.3 — Debug overlays →](03-debug-overlays.md)

---

## Debug output: seeing the internals

The most basic dev facility is **debug output** — printing internal state so the developers can *see* it. `speed.exe`
has two channels:

- **To the debugger** — `DebugPrint`/`DebugStringA` write to the debug output stream (the IDE's output window,
  [C58.1](../C58-Build-Pipeline/01-shipping-exe.md)) — the classic `printf`-debugging log.
- **To the screen** — `ScreenPrintf`/`DebugScreenMessage` print *on the game screen* (via the `ScreenPrintf.fng`
  overlay, [Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) — so state is visible *while playing*, without a
  debugger attached.

So developers could log values two ways: to the debugger (detailed, offline) or on-screen (live, in-game). The
on-screen channel (`ScreenPrintf`) is especially useful for a *game* — you can watch a value (a car's speed, an AI
state) *as you play*, overlaid on the action. That both channels shipped ([C67.1](01-debug-in-shipped.md)) shows the
codebase was *instrumented for observation* — the developers could see whatever they printed.

> ✅ *Verified:* `DebugPrint`, `DebugStringA`, `ScreenPrintf`, `DebugScreenMessage` are present in `speed.exe` — the
> debug output channels; `ScreenPrintf.fng` is the on-screen overlay ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)).

## Assertions: checking invariants

The other core facility is **assertions** — `Assert`/`Assertion` — checks that a condition *must* hold, breaking
(`DebugBreak`) into the debugger if it doesn't:

```cpp
Assert(pBody != nullptr);          // this must never be null here
Assert(index < poolSize);          // bounds must hold
Assert(state == EXPECTED_STATE);   // the state machine must be here
```

An assertion is a *documented invariant* — a fact the developers *knew must be true* at that point, encoded as a
runtime check. When an assertion *fails*, it means a bug violated an assumption — and `DebugBreak` drops the
developer into the debugger at the exact spot. So assertions are the developers' *safety net*: they catch bugs *at
the moment an invariant breaks*, not later when the symptom appears. That the codebase is *littered* with
assertions ([C67.1](01-debug-in-shipped.md)) shows a *defensive* engineering culture — the developers encoded their
assumptions as checks, so violations surfaced immediately.

> ✅ *Verified:* `Assert`, `Assertion`, `asserts`, and `DebugBreak` are present in `speed.exe` — the assertion
> system.

## Assertions as documentation

For RE, assertions are *especially* valuable — each is a *documented fact about the code*
([C67.1](01-debug-in-shipped.md)):

- **An asserted invariant is a known truth.** `Assert(index < poolSize)` tells you *there's a pool with a size, and
  the index must be within it* — structural facts ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md))
  the developers documented via the check.
- **The assertion message** (often a string) *names* the invariant — a human-readable statement of what must hold.
- **The location** of an assert marks a *critical point* — the developers cared enough to check there.

So assertions are the developers' *annotations* of their own code — statements of what must be true, embedded in the
binary. Reading them ([C67.5](05-reading-debug.md)) recovers *documented invariants* — facts the book can cite with
confidence ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)), because the developers *asserted* them.
This is the deepest value of the shipped debug tooling: it's not just tools, it's *the developers' own
documentation of their code's rules*, preserved in the retail binary. An assertion is a fact the developers
guaranteed — as good as a comment, and it shipped.

## Why on-screen debug matters

The **on-screen** debug channel (`ScreenPrintf`/`DebugScreenMessage`) deserves emphasis — it's characteristic of
*game* development:

- **Live observation.** A game runs in real time; a developer needs to see state *while it happens* (the car
  drifting, the AI deciding). On-screen debug shows it *live*, overlaid on the action
  ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)).
- **No debugger needed.** On a console or a running build, attaching a debugger is heavy; on-screen text is
  lightweight — glance at the number, keep playing.
- **The FEng overlay.** `ScreenPrintf.fng` ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) is a *front-end
  package* — the debug text uses the *same* FEng system ([C65.1](../C65-HUD-Runtime/01-hud-runtime.md)) as the HUD,
  so debug output is drawn like any UI.

So on-screen debug is the *game developer's printf* — live, overlaid, lightweight. That it's built on FEng
([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) shows the *uniformity* of the engine: even debug text is a
front-end widget. Reading `ScreenPrintf` confirms the developers watched the game's internals *live on screen* while
building it — the most immediate form of the observation the debug tooling provides.

## RE implications

- **Debug output** — `DebugPrint`/`DebugStringA` (to the debugger), `ScreenPrintf`/`DebugScreenMessage` (on
  screen) — the developers' eyes.
- **Assertions** (`Assert`/`Assertion`, break via `DebugBreak`) check invariants — a defensive culture, catching
  bugs at the violation.
- **Assertions are documentation** — each a documented invariant, a fact the book can cite.
- **On-screen debug** (`ScreenPrintf.fng`) is the game-dev printf — live, FEng-based
  ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)).

---

### Key takeaways

- **Debug output** has two channels — **`DebugPrint`/`DebugStringA`** (to the debugger) and **`ScreenPrintf`/
  `DebugScreenMessage`** (on screen) — the developers' *eyes* into the internals.
- **Assertions** (`Assert`/`Assertion`, breaking via `DebugBreak`) check **invariants** — documented facts the
  developers knew must hold — a **defensive** engineering culture catching bugs at the violation.
- **Assertions are documentation** — each encodes a *known truth* about the code (a pool size, a state); reading
  them recovers **documented invariants** the book can cite.
- **On-screen debug** (`ScreenPrintf.fng`) is the **game-dev printf** — live, overlaid, lightweight — and built on
  the **same FEng system** as the HUD ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)).
- The debug I/O shows the codebase was **instrumented for observation** — the developers could see, and check,
  whatever mattered.

**Continue:** [C67.3 — Debug overlays](03-debug-overlays.md) · [Chapter 67 hub](C67-Debug-Facilities.md)
