# C46.4 — The Override Goals

> **The one-sentence version:** four goals carry their own large vtables where real behaviour is coded —
> `AIGoalFleePursuit` (94 methods, intelligent evasion), `AIGoalHeliPursuit` (90, air tracking), `AIGoalHeliExit`
> (86, departure), `AIGoalRacer` (82, racing brain) — the AIs with genuine intelligence.

[← C46.3 — The data-only goals](03-data-only-goals.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md) ·
[Next: C46.5 — The action menu →](05-action-menu.md)

---

## The four exceptions

Where ten goals are thin data-only specialisations ([C46.3](03-data-only-goals.md)), **four carry their own large
override vtables** — real, complex code. All verified by method count:

| Goal | vtable | Methods | What the code is |
|---|---|---|---|
| `AIGoalFleePursuit` | `0x00892D00` | **94** | intelligent evasion — using traffic, routes, distance |
| `AIGoalHeliPursuit` | `0x00892D10` | **90** | airborne tracking, spotlight, ground coordination |
| `AIGoalHeliExit` | `0x00892D20` | **86** | flying the chopper out cleanly |
| `AIGoalRacer` | `0x00892D30` | **82** | racing intelligence — line, drafting, catch-up |

Compared to the shared base's **12 methods** ([C46.3](03-data-only-goals.md)), these are 7–8× larger — they
override the base's behaviour with substantial custom logic. These are the four AIs whose intent is too
sophisticated to express as a menu of shared actions.

> ✅ *Verified:* the four override vtables are confirmed by pointer count in `speed.exe` — `AIGoalFleePursuit`
> `0x00892D00`/**94**, `AIGoalHeliPursuit` `0x00892D10`/**90**, `AIGoalHeliExit` `0x00892D20`/**86**, `AIGoalRacer`
> `0x00892D30`/**82**. All four exceed the 12-method data-only base ([C46.3](03-data-only-goals.md)) many times
> over.

## FleePursuit: the most-overridden

`AIGoalFleePursuit` (**94 methods** — the most) is the goal of a car **evading** a pursuit — an AI racer or rival
running from cops ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)). That it's the *most-overridden* goal is
telling: **evading intelligently is harder than chasing.** A pursuer just needs to head toward the target; an
evader must:

- **Plan routes** — pick escape paths through the road network ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md))
  that lose pursuers.
- **Use traffic and terrain** — weave through traffic, use cover, exploit shortcuts.
- **Manage distance and line-of-sight** — break contact, then stay broken, to cool the pursuit
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **React to threats** — dodge roadblocks, spike strips, and rams.

All of that is real intelligence — 94 methods of it. This is the AI you race *against* when a rival flees, and it's
the game's hardest AI problem (a good evader is what makes a chase feel like a contest). The method count is the
measure of that difficulty.

## Racer: the racing brain

`AIGoalRacer` (**82 methods**) is the **circuit-opponent intelligence** — the brain of every non-pursuit race
opponent. Its override code is racing skill:

- **Line choice** — following and optimising the racing line ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)).
- **Rubber-banding / catch-up** — the tuned tendency to keep the race close (speed up when behind, ease when ahead)
  — a core of arcade-racer AI.
- **Drafting and overtaking** — using slipstream, choosing where to pass.

This is what distinguishes a *race opponent* from a *cop* ([C46.3](03-data-only-goals.md)): a cop just chases you,
but a racer must drive a *good race* — fast, clean, and competitive. The 82 methods are the racing craft that makes
circuit opponents feel like drivers rather than pursuers.

## The helicopter goals

`AIGoalHeliPursuit` (**90**) and `AIGoalHeliExit` (**86**) are the **helicopter's** intents
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) — and they override heavily because
*flying* is fundamentally different from *driving*:

- **`AIGoalHeliPursuit`** — airborne tracking (following you from above, not on roads), spotlight aiming, and
  coordination with ground units and the air-support strategy
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)). The chopper's whole job.
- **`AIGoalHeliExit`** — flying the chopper out cleanly when the pursuit ends or it's dismissed — a controlled
  departure.

The helicopter can't reuse the ground actions (Race, Ram, PursuitOffRoad, [C46.5](05-action-menu.md)) — it flies —
so its goals must supply their own flight behaviour, hence the large vtables. This is why the helicopter goals
override rather than share the base ([C46.3](03-data-only-goals.md)): a flying unit needs different code, not a
different menu of ground actions.

## Where the AI's complexity lives

The override goals are a map of **where the AI's real code complexity lives**:

- **Evasion** (`FleePursuit`, 94) — the hardest, because a smart evader must out-think pursuers.
- **Flight** (`HeliPursuit` 90, `HeliExit` 86) — a whole different locomotion.
- **Racing** (`Racer`, 82) — competitive driving craft.

Everything else — the cops' pursuit tactics ([C46.3](03-data-only-goals.md)) — is *composed* from shared actions
and thin data-only goals. So the engine spends its AI code budget on the four genuinely-hard problems and gets the
rest for free through composition. This is a clear-eyed engineering allocation: write real code only where the
behaviour truly demands it, and make everything else data. Reading the method counts ([C46.6](06-reading-ai.md))
tells you exactly where that line falls.

## RE implications

- **Four goals override** with large vtables (verified) — `FleePursuit` (94), `HeliPursuit` (90), `HeliExit` (86),
  `Racer` (82).
- **FleePursuit is the most-overridden** — intelligent evasion is the hardest AI problem (route-planning, using
  traffic, breaking contact).
- **Racer is the racing brain** — line, rubber-banding, drafting — what makes a race opponent, not a cop.
- **The helicopter goals** override because flight ≠ driving — they can't reuse ground actions.

---

### Key takeaways

- **Four goals carry large override vtables** (verified): `AIGoalFleePursuit` (**94** methods), `AIGoalHeliPursuit`
  (**90**), `AIGoalHeliExit` (**86**), `AIGoalRacer` (**82**) — vs. the 12-method data-only base.
- **`FleePursuit` (94, the most)** is intelligent evasion — harder than chasing, because a smart evader must
  out-think pursuers with routes, traffic, and distance.
- **`Racer` (82)** is the racing brain — line choice, rubber-banding/catch-up, drafting — what makes a *race
  opponent* distinct from a cop.
- **The helicopter goals** override because **flight ≠ driving** — they need their own flight behaviour, not ground
  actions.
- The override goals **map where the AI's real complexity lives** — evasion, flight, racing — while cop tactics are
  composed for free from shared actions.

**Continue:** [C46.5 — The action menu](05-action-menu.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md)
