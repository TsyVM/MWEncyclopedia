# C48.5 — Reading Pursuit in RE

> **The one-sentence version:** navigate the pursuit by `AIPursuit` (the director, 98 methods), the Heat dispatch
> tables (`CopCountRecord`/`CopFormationRecord`, ×22 each), the lifecycle event strings (`PursuitBegins` →
> `Busted`/`PursuitOver`), and the byte-verified bust envelope (`0x443BA0`, radii 15/90, hold 3.0 s).

[← C48.4 — The bust & the evade](04-bust-evade.md) · [Chapter 48 hub](C48-Pursuit-Heat.md) ·
[Next: Chapter 49 — Cops: Dispatch, Formations, Roadblocks & Bust →](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)

---

## Anchors for pursuit RE

The pursuit is anchored on verified structures:

- **`AIPursuit`** (`0x00892770`, 98 methods) — the chase director ([C48.1](01-the-cast.md)).
- **The dispatch tables** — `CopCountRecord` (×22), `CopFormationRecord` (×22) ([C48.2](02-heat.md)).
- **The lifecycle events** — `PursuitBegins`, `PursuitApproaching`, `PursuitEscalation`, `PursuitAddsCar`/`AddsHeli`/
  `AddsRoadblock`, `PursuitEnds`, `PursuitOver`, `Busted` ([C48.3](03-escalation-ladder.md)).
- **The bust envelope** — perp tick `0x443BA0`, `BustSpeed` (`0x769E8D9E`), radii 15/90, gauge 5.0, hold 3.0 s
  ([C48.4](04-bust-evade.md)).

From these, the whole pursuit is navigable: the director, the tables it indexes, the lifecycle, and the resolution.

## The RE workflow

Reading the pursuit:

1. **Find the director** — `AIPursuit` ([C48.1](01-the-cast.md)); its 98 methods are the state machine.
2. **Read the Heat tables** — `CopCountRecord`/`CopFormationRecord` ([C48.2](02-heat.md)) in the vault; the rows
   are the per-Heat difficulty.
3. **Trace the lifecycle** — the `Pursuit*` events ([C48.3](03-escalation-ladder.md)) mark the state transitions.
4. **Decode the bust** — the envelope constants ([C48.4](04-bust-evade.md)) at their `.rdata` addresses; the perp
   tick `0x443BA0` evaluates them.

The output is the full pursuit picture: the director, the difficulty tables, the lifecycle, and the resolution
math.

## Data-over-code, verified

The pursuit is the clearest example in the whole engine of the **data-over-code** design
([C48.2](02-heat.md)–[C48.3](03-escalation-ladder.md)), and RE proves it:

- **The code is a small, fixed set** — `AIPursuit` (98 methods), the goals/actions
  ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)), the bust envelope
  ([C48.4](04-bust-evade.md)). This is the machinery.
- **The behaviour is data** — the Heat tables (`CopCountRecord`/`CopFormationRecord`, ×22 each), the escalation
  thresholds (`pursuit` vault), `BustSpeed` per Heat. This is the *difficulty and drama*.

So to change how pursuits *feel* — harder, faster-escalating, more cops — you edit the *data*, not the code
([C48.3](03-escalation-ladder.md)). The verified split (a handful of code classes, hundreds of vault references)
*is* the proof: `CopCountRecord`/`CopFormationRecord` at ×22 each, `BustSpeed` a per-Heat field, the goals
vault-tuned ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)). The pursuit is a data-driven system on a
verified code skeleton — the industry pattern of a designer-tunable AI system, cleanly realised.

## Verifying a pursuit claim

Every pursuit claim reduces to a check:

- **"How many cops at Heat N?"** — read the `CopCountRecord` row N ([C48.2](02-heat.md)).
- **"When do cops start ramming?"** — the goal-swap thresholds in the `pursuit` vault
  ([C48.3](03-escalation-ladder.md)); `AIGoalPit` is vault-tuned (×9).
- **"What's the bust radius?"** — 15.0 (`[0x890DAC]`), or 90.0 engaged (`[0x892FA8]`) ([C48.4](04-bust-evade.md)).
- **"Is there a cop vision cone?"** — no; the bust is a distance/speed/time envelope
  ([C48.4](04-bust-evade.md)) — verified, no raycast in the bust path.

Each is a table row, a vault field, or a `.rdata` constant — all checkable, all checked. The pursuit's reputation
as "the deep system" belies how *legible* it is under RE: a director class, two dispatch tables, a lifecycle of
named events, and a three-number bust envelope.

## Pursuit is AIPursuit conducting the data

The one-line summary of the pursuit: **`AIPursuit` conducts the data.** The director
([C48.1](01-the-cast.md)) reads Heat, indexes the dispatch tables ([C48.2](02-heat.md)), walks the escalation
ladder ([C48.3](03-escalation-ladder.md)), and resolves via the bust envelope ([C48.4](04-bust-evade.md)) — all
over a body of vault data that defines the *difficulty and drama*. The code is the conductor; the data is the
score. This is why the pursuit is both Most Wanted's signature *and* its most tunable system — it's a small,
verified engine playing a large, editable score. The next chapter
([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) reads the *fleet* side — the cop
types, formations, roadblocks, and the dispatch that supplies the pursuit.

## RE implications

- **Anchor on** `AIPursuit`, the dispatch tables, the lifecycle events, and the bust envelope constants.
- **The RE workflow** — director → Heat tables → lifecycle → bust math.
- **Data-over-code, verified** — a small code skeleton, behaviour in the vault (×22 tables, per-Heat `BustSpeed`).
- **Every claim reduces to a check** — a table row, a vault field, or a `.rdata` constant.

---

### Key takeaways

- The pursuit is anchored on **`AIPursuit`** (director), the **dispatch tables** (`CopCountRecord`/
  `CopFormationRecord`, ×22), the **lifecycle events** (`PursuitBegins` → `Busted`/`PursuitOver`), and the **bust
  envelope** constants.
- The RE workflow: **director → Heat tables → lifecycle → bust math**.
- The pursuit is the engine's clearest **data-over-code** system — a small verified code skeleton (`AIPursuit`,
  goals, the envelope) playing a large editable score (the vault tables and thresholds).
- Every pursuit claim reduces to a **table row, vault field, or `.rdata` constant** — all checkable; the "deep
  system" is remarkably legible.
- **`AIPursuit` conducts the data** — the code is the conductor, the vault is the score; MW's signature system is
  also its most tunable.

**Next:** [Chapter 49 — Cops: Dispatch, Formations, Roadblocks & Bust](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md):
the fleet that supplies the pursuit.

**Sources:** `speed.exe` (verified: `AIPursuit` `0x00892770`/98; pursuit lifecycle strings `PursuitBegins`/
`PursuitAddsCar`/`AddsHeli`/`AddsRoadblock`/`PursuitEnds`/`Busted`; bust envelope — perp tick `0x443BA0`,
`rh("BustSpeed")=0x769E8D9E`, radii `15.0` `[0x890DAC]`/`90.0` `[0x892FA8]`, gauge `5.0` `[0x890DA4]`, hold `3.0`
`[0x8EB318]`; `MPerpBusted`/`MPerpEscaped`/`HELICOPTER_LINE_OF_SIGHT`/`AIRacerBusted`); `GLOBAL/attributes.bin`
(verified: `CopCountRecord` `0xFCAA46E2` ×22, `CopFormationRecord` `0xB5A53D76` ×22, cop-type keys `copsuv` ×16,
`copheli` ×15, `copcross` ×7).
