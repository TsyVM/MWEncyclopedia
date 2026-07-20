# C46.2 — The Goal Catalogue

> **The one-sentence version:** the 15 verified `AIGoal*` classes cover every AI role — cop (Pursuit, Pit, Ram,
> PullOver, HeadOnRam, StopShort, StaticRoadBlock, Patrol), helicopter (HeliPursuit, HeliExit, HeliRoadBlock),
> racer/suspect (Racer, FleePursuit), traffic (Traffic), and the inert None.

[← C46.1 — The goal/action model](01-goals-and-actions.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md) ·
[Next: C46.3 — The data-only goals →](03-data-only-goals.md)

---

## The 15 goals

Every AI-driven car in Most Wanted has one of these goals as its intent
([C46.1](01-goals-and-actions.md)), verified as `AIGoal*` strings in `speed.exe`:

| Goal | Role | Intent |
|---|---|---|
| `AIGoalPursuit` | cop | the default chase — race the line after the target |
| `AIGoalPit` | cop | execute a PIT manoeuvre — spin the target out |
| `AIGoalRam` | cop | sustained ramming — shunt the target |
| `AIGoalPullOver` | cop | box and stop the target — pre-bust pressure |
| `AIGoalHeadOnRam` | cop | intercept head-on — meet the target oncoming |
| `AIGoalStopShort` | cop | brake-check — pull ahead and stop the target short |
| `AIGoalStaticRoadBlock` | cop | hold a roadblock slot |
| `AIGoalPatrol` | cop | idle patrol — drive as traffic until a stimulus |
| `AIGoalHeliPursuit` | heli | the helicopter's chase — track from the air |
| `AIGoalHeliExit` | heli | the helicopter's departure — fly out cleanly |
| `AIGoalHeliRoadBlock` | heli | air-coordinated roadblock support |
| `AIGoalRacer` | racer | win the race — the circuit-opponent brain |
| `AIGoalFleePursuit` | suspect | flee intelligently — the racer/rival evading |
| `AIGoalTraffic` | traffic | be civilian traffic — pure lane-following |
| `AIGoalNone` | — | no intent — the inert default |

> ✅ *Verified:* all 15 `AIGoal*` strings are present in `speed.exe`. The tactical ones are vault-tuned:
> `rh("AIGoalPit")=0xEC619CB2` ×9 and `rh("AIGoalStaticRoadBlock")=0x9E55B2E3` ×11 in `attributes.bin`;
> `AIGoalRacer` ×1.

## Four families of intent

The goals cluster into four role families, each a kind of AI driver:

**Cop goals (8).** The largest family — the pursuit behaviours ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)):
from the passive `AIGoalPatrol` (drive around until provoked), through the baseline `AIGoalPursuit` (chase), up the
aggressive ladder `AIGoalPit` → `AIGoalRam` → `AIGoalPullOver` → `AIGoalHeadOnRam` → `AIGoalStopShort` (make
contact, spin, box, intercept, brake-check), plus `AIGoalStaticRoadBlock` (hold a block). The *escalation of a
pursuit* is the manager ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) walking a cop up this
ladder as Heat rises.

**Helicopter goals (3).** The air unit ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
— `AIGoalHeliPursuit` (track from above), `AIGoalHeliExit` (leave), `AIGoalHeliRoadBlock` (coordinate a block from
the air). These pair with the helicopter physics ([C41.6](../C41-Physics-RigidBody/06-vehicle-types.md)).

**Racer/suspect goals (2).** `AIGoalRacer` (win a race — the circuit-opponent intelligence) and `AIGoalFleePursuit`
(evade — used by AI racers and rivals running from cops). These are the two *most sophisticated* goals
([C46.4](04-override-goals.md)) — racing and evading well are hard.

**Traffic (1) + None (1).** `AIGoalTraffic` is every ambient car's intent (lane-follow); `AIGoalNone` is the inert
placeholder before a real goal is assigned.

## The pursuit escalation ladder

The cop goals form an **escalation ladder** — the heart of the pursuit ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)):

```
Patrol → Pursuit → Pit / Ram / PullOver / HeadOnRam / StopShort
(passive) (chase)   (─────────── aggressive contact ───────────)
```

As Heat rises ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), the manager
([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)) promotes cops from just chasing (`Pursuit`) to
actively trying to stop you (`Pit`, `Ram`, `PullOver`) — authorising contact and tactics. That the tactical goals
`AIGoalPit` (×9) and `AIGoalStaticRoadBlock` (×11) are the vault-tuned ones ([above](#the-15-goals)) is telling:
these are the behaviours with *parameters* (how aggressively to PIT, how to space a roadblock), so they carry vault
data, while the basic intents (Patrol, Pursuit) are pure code. The escalation is thus both a goal swap *and* a
shift into more-tuned, more-specific behaviour.

> 🟡 *Reasoned:* the mapping of goals to the Heat-driven escalation ladder is the pursuit design
> ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)); the exact Heat thresholds are vault data. The goal set
> and the vault-tuning of `Pit`/`StaticRoadBlock` are verified.

## Why name every intent

Making each intent a *named class* (rather than an integer state) is characteristic of the class-driven engine
([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)):

- **Each goal is a first-class object** — constructed via the factory
  ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)), registered on the goal list-head
  `0x0090D8E8`, with a vtable ([C46.3](03-data-only-goals.md)–[C46.4](04-override-goals.md)).
- **The name is the identity** — `AIGoalPit` *is* the PIT intent, legible in the code and (hashed) in the vault
  ([above](#the-15-goals)).
- **Enumeration is possible** — the goal family list ([C46.6](06-reading-ai.md)) can be walked to see every intent
  the AI can hold.

So the goal catalogue isn't a hidden enum — it's a set of named, registered classes, which is why we can recover it
exactly (grep `AIGoal*`) and reason about each. The naming is what makes the AI's repertoire *readable* — you can
see, from the strings alone, that cops can PIT, ram, box, intercept, brake-check, and block.

## RE implications

- **15 named goals** cover four role families — cop (8), helicopter (3), racer/suspect (2), traffic + none.
- **The cop goals form an escalation ladder** — Patrol → Pursuit → aggressive (Pit/Ram/PullOver/HeadOnRam/StopShort).
- **The tactical goals are vault-tuned** — `AIGoalPit` ×9, `AIGoalStaticRoadBlock` ×11 — the behaviours with
  parameters.
- **Each intent is a named class** — recoverable by grep, registered on `0x0090D8E8`, legible.

---

### Key takeaways

- The **15 verified goals** cover every AI role: **cop** (Pursuit, Pit, Ram, PullOver, HeadOnRam, StopShort,
  StaticRoadBlock, Patrol), **helicopter** (HeliPursuit, HeliExit, HeliRoadBlock), **racer/suspect** (Racer,
  FleePursuit), **traffic**, and **None**.
- The **cop goals form the pursuit escalation ladder** — from passive Patrol, to chasing Pursuit, up to aggressive
  contact (Pit/Ram/PullOver/HeadOnRam/StopShort) as Heat rises.
- The **tactical goals are the vault-tuned ones** (`AIGoalPit` ×9, `AIGoalStaticRoadBlock` ×11) — they carry
  parameters; basic intents are pure code.
- **Racer and FleePursuit** are the most sophisticated ([C46.4](04-override-goals.md)) — winning and evading well
  are hard.
- Each intent is a **named, registered class** — the AI's whole repertoire is recoverable by grepping `AIGoal*`.

**Continue:** [C46.3 — The data-only goals](03-data-only-goals.md) · [Chapter 46 hub](C46-AI-Goals-Actions.md)
