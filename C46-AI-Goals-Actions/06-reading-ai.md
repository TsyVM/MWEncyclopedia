# C46.6 — Reading AI in RE

> **The one-sentence version:** navigate the AI by the goal/action strings (`AIGoal*`/`AIAction*`), the goal
> list-head `0x0090D8E8`, the shared base vtable `0x00892B20` (12 methods, the data-only goals), and the four
> override vtables — reading the AI as thin data goals over reusable actions, with real code only where it's
> needed.

[← C46.5 — The action menu](05-action-menu.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md) ·
[Next: Chapter 47 — AI Driver Brain & Vehicle Hierarchy →](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)

---

## Anchors for AI RE

The AI is anchored on verified structures:

- **The goal/action strings** — grep `AIGoal*` (15) and `AIAction*` (17) ([C46.2](02-goal-catalog.md),
  [C46.5](05-action-menu.md)) — the whole repertoire.
- **The goal list-head `0x0090D8E8`** — where goals register (runtime-populated).
- **The shared base vtable `0x00892B20`** (12 methods) — the data-only goals ([C46.3](03-data-only-goals.md)).
- **The four override vtables** — `FleePursuit` `0x00892D00` (94), `HeliPursuit` `0x00892D10` (90), `HeliExit`
  `0x00892D20` (86), `Racer` `0x00892D30` (82) ([C46.4](04-override-goals.md)).
- **The vault tuning** — `AIGoalPit` ×9, `AIGoalStaticRoadBlock` ×11 ([C46.2](02-goal-catalog.md)).

From these, the AI is fully navigable: the repertoire (strings), the structure (vtables), and the tuning (vault).

## The RE workflow

Reading the AI:

1. **Enumerate goals and actions** — grep `AIGoal*`/`AIAction*` ([C46.2](02-goal-catalog.md),
   [C46.5](05-action-menu.md)); you have the whole behaviour vocabulary.
2. **Sort by vtable** — which goals share `0x00892B20` (data-only, [C46.3](03-data-only-goals.md)) vs. which
   override ([C46.4](04-override-goals.md)); the method counts show where the real code is.
3. **Recover the menus** — each data-only goal's constructor installs its action menu
   ([C46.3](03-data-only-goals.md)); read it to know the goal's repertoire.
4. **Find the tuning** — the vault-keyed goals ([C46.2](02-goal-catalog.md)) and the manager's swap thresholds
   ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).

The output is the full AI picture: the repertoire, which goals are data vs. code, their menus, and their tuning.

## The method-count map

The single most illuminating RE artifact for the AI is the **method-count map** — sorting the goals by their vtable
method count reveals the whole architecture at a glance:

```
12   ← shared base (0x00892B20): Pursuit, Pit, Ram, PullOver, HeadOnRam,
       StopShort, Patrol, Traffic, StaticRoadBlock, None   (DATA-ONLY)
82   ← AIGoalRacer      (racing brain)
86   ← AIGoalHeliExit   (chopper departure)
90   ← AIGoalHeliPursuit(chopper chase)
94   ← AIGoalFleePursuit(intelligent evasion — the most)   (OVERRIDE)
```

The gulf between 12 and 82–94 *is* the architecture: ten goals costing 12 methods total (shared), four goals
costing 82–94 each. The AI's code complexity is concentrated in exactly four places — evasion, flight×2, racing —
and everything else is composition ([C46.3](03-data-only-goals.md)). Reading this map, you know instantly where to
spend RE effort (the four overrides) and what's data (the ten shared). This is verification-first RE at its most
efficient: the method counts, cheaply obtained ([C46.6](#verifying-a-goal)), map the whole system's complexity.

## Verifying a goal

Verifying a claim about a goal reduces to checks:

- **"Is X a goal?"** — grep `speed.exe` for `AIGoalX` ([C46.2](02-goal-catalog.md)).
- **"Is it data-only?"** — is its vtable `0x00892B20` (12 methods)? ([C46.3](03-data-only-goals.md))
- **"Is it an override?"** — does it have its own vtable with many methods? (count the pointers,
  [C46.4](04-override-goals.md))
- **"Is it tuned?"** — does `rh("AIGoalX")` appear in `attributes.bin`? (`Pit` ×9, `StaticRoadBlock` ×11,
  [C46.2](02-goal-catalog.md))

Each is a grep, a pointer count, or a hash lookup — the verification-first discipline
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)). The AI is especially amenable to it
because it's *named* (goals/actions are classes) and *structured* (vtables), so every claim has a concrete check.

## The AI's design in one line

The AI's whole design, recovered from RE: **a small set of reusable actions, composed into intents (goals) that are
mostly pure data, swapped by managers on tuned thresholds, with real code only for the four genuinely-hard
behaviours (evasion, flight, racing).** This is a model of pragmatic game-AI engineering — maximal behaviour from
minimal code, maximal tunability through data. The next chapter ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md))
covers the *managers* that swap the goals and the AI *vehicle* classes that host them, completing the driver brain.

## RE implications

- **Anchor on** the `AIGoal*`/`AIAction*` strings, the list-head `0x0090D8E8`, the shared base `0x00892B20`, and
  the four override vtables.
- **The RE workflow** — enumerate → sort by vtable → recover menus → find tuning.
- **The method-count map** reveals the architecture — 12 (shared) vs. 82–94 (four overrides); complexity is
  concentrated.
- **Every AI claim reduces to a check** — grep, pointer count, or hash lookup.

---

### Key takeaways

- The AI is anchored on the **`AIGoal*`/`AIAction*` strings**, the **goal list-head `0x0090D8E8`**, the **shared
  base vtable `0x00892B20`** (12 methods), and the **four override vtables**.
- The RE workflow: **enumerate goals/actions → sort by vtable (data-only vs. override) → recover the menus → find
  the vault tuning**.
- The **method-count map** (12 shared vs. 82–94 for four overrides) reveals the whole architecture — complexity
  concentrated in evasion, flight×2, and racing.
- Every AI claim reduces to a **grep, pointer count, or hash lookup** — the AI is especially verifiable because
  it's named and structured.
- The AI's design: **reusable actions composed into mostly-data goals, swapped by managers on tuned thresholds,
  with real code only where behaviour truly demands it.**

**Next:** [Chapter 47 — AI Driver Brain & Vehicle Hierarchy](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md): the
managers and AI vehicle classes.

**Sources:** `speed.exe` (verified: 15 `AIGoal*` + 17 `AIAction*` strings; shared base vtable `0x00892B20`/12
methods; override vtables `AIGoalFleePursuit` `0x00892D00`/94, `AIGoalHeliPursuit` `0x00892D10`/90, `AIGoalHeliExit`
`0x00892D20`/86, `AIGoalRacer` `0x00892D30`/82; goal list-head `0x0090D8E8`); `GLOBAL/attributes.bin` (verified:
`AIGoalPit` ×9, `AIGoalStaticRoadBlock` ×11, `AIGoalRacer` ×1 as reflection-hash keys).
