# C54.4 — Bounty & Milestones

> **The one-sentence version:** progression up the Blacklist is gated by two systems — Bounty (`BountyDatum`), the
> notoriety you earn from pursuits and events, and Milestones (`MilestoneAtStake`, `MilestoneProgress`,
> `MilestoneRewards`), the specific challenges you must complete to unlock a rival's showdown.

[← C54.3 — The Blacklist](03-the-blacklist.md) · [Chapter 54 hub](C54-GameFlow-Blacklist.md) ·
[Next: C54.5 — Reading GameFlow in RE →](05-reading-gameflow.md)

---

## Bounty: the currency of notoriety

**Bounty** (`BountyDatum`) is the *currency of the Blacklist* — a measure of your notoriety, earned by being a
menace to the city:

- **Earned in pursuits** — evading cops ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), causing damage,
  performing pursuit feats (dodged roadblocks, [C49.4](../C49-Cops-Dispatch-Roadblocks/04-roadblocks.md); disabled
  cops, [C49.5](../C49-Cops-Dispatch-Roadblocks/05-spikes-breakers.md)) all earn bounty.
- **Earned in events** — winning races ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) adds bounty.
- **Accumulates toward the next rival** — each Blacklist rival ([C54.3](03-the-blacklist.md)) requires a bounty
  threshold before you can challenge them.

So bounty is the *reward for being wanted* — the more trouble you cause and races you win, the more notorious you
become, and notoriety is what earns you a shot at the next rival. This ties the *pursuit* system
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) directly to *progression*: the chases aren't a side activity
— surviving and dominating them *is* how you advance. The bigger the pursuit you escape, the more bounty, the closer
the next Blacklist challenge.

> ✅ *Verified:* `Bounty` and `BountyDatum` are present in `speed.exe` — the notoriety/reward system. It's earned in
> pursuits ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) and events, and gates Blacklist progression
> ([C54.3](03-the-blacklist.md)).

## Milestones: the specific challenges

Where bounty is a *threshold*, **Milestones** are *specific challenges* — the concrete tasks that unlock a rival's
showdown. The verified milestone system is rich:

| Class | Role |
|---|---|
| `Milestone` / `Milestones` | the challenge objectives for a rival |
| `MilestoneAtStake` | the milestone currently being attempted |
| `MilestoneProgress` | how far toward completing it |
| `MilestoneReached` | a completed milestone |
| `MilestoneRewards` | what completing it grants |
| `MilestonesScreen` | the UI showing your milestones |
| `MilestoneBoard` / `MilestoneDatum` | the milestone data/display |

So each Blacklist rival ([C54.3](03-the-blacklist.md)) has a set of **milestones** — e.g. "win 3 of these races,"
"survive a Heat-4 pursuit for N minutes," "reach a bounty total." Completing them (`MilestoneReached`) unlocks the
rival's challenge, and grants `MilestoneRewards`. `MilestoneProgress` tracks your advancement; the
`MilestonesScreen` shows what's left. Milestones are thus the *checklist* to the next rival — concrete goals that
structure your climb between the boss encounters.

## Bounty + milestones: the two gates

Together, bounty and milestones form the **two-part gate** on each Blacklist rung ([C54.3](03-the-blacklist.md)):

```
to challenge Blacklist rival #N:
   1. Bounty threshold reached   (BountyDatum ≥ rival's requirement)  ← notoriety
   2. Milestones completed        (MilestoneReached for the rival's set) ← specific tasks
   → the rival's challenge unlocks
```

The *bounty* gate ensures you've been *active and notorious enough* (a general measure); the *milestone* gate
ensures you've completed *specific accomplishments* (concrete tasks). Requiring both paces the game well: you can't
grind one race for bounty and skip ahead, nor complete tasks without earning notoriety — you must do a *variety* of
things (win races, survive pursuits, hit bounty) to advance. This dual gate is what gives Most Wanted its balanced
progression: a mix of racing ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) and pursuit
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), measured both broadly (bounty) and specifically
(milestones).

> 🟡 *Reasoned:* the two-gate (bounty threshold + milestone completion) structure is Most Wanted's documented
> progression design, consistent with the verified `Bounty`/`Milestone*` classes; the exact per-rival requirements
> are career vault data. The bounty and milestone systems are verified.

## Why gate progression

Gating Blacklist progression behind bounty and milestones (rather than just "win the next race") serves the game's
design ([C54.3](03-the-blacklist.md)):

- **It paces the climb.** The gates space out the boss encounters, so each rival feels *earned* — you've done the
  work (races, pursuits) to deserve the shot.
- **It rewards all activities.** Both racing *and* pursuit feed progression (races → bounty + milestones; pursuits →
  bounty) — so the whole game, not just one mode, advances you.
- **It gives concrete goals.** Milestones ([above](#milestones-the-specific-challenges)) turn "get better" into a
  *checklist* — the player always has specific, achievable next steps toward the rival.

So bounty and milestones are the *pacing and goal* systems of the career — they turn the Blacklist
([C54.3](03-the-blacklist.md)) from a raw difficulty ladder into a *structured progression* with clear requirements
and rewards at each step. They're the machinery that makes the climb feel *fair* (earned) and *guided* (clear next
steps) — the connective tissue between the events you play and the rivals you climb toward.

## RE implications

- **Bounty** (`BountyDatum`) is earned notoriety — from pursuits and events — gating each Blacklist rung.
- **Milestones** (`MilestoneAtStake`/`Progress`/`Reached`/`Rewards`) are specific challenges unlocking a rival's
  showdown.
- **Two-part gate** — bounty threshold + milestone completion — paces the climb and rewards all activities.
- **Gating progression** makes the climb earned, varied, and guided (concrete next steps).

---

### Key takeaways

- **Bounty** (`BountyDatum`) is the **currency of notoriety** — earned from pursuits (evading, feats) and events,
  accumulating toward each Blacklist rival's threshold.
- **Milestones** (`MilestoneAtStake`, `MilestoneProgress`, `MilestoneReached`, `MilestoneRewards`) are the
  **specific challenges** — win N races, survive a pursuit — that unlock a rival's showdown.
- Each Blacklist rung is a **two-part gate** — a **bounty threshold** (general notoriety) *and* **milestone
  completion** (concrete tasks) — before the rival's challenge unlocks.
- The dual gate **paces the climb** and **rewards all activities** — both racing and pursuit advance you, measured
  broadly and specifically.
- Bounty and milestones are the **pacing and goal machinery** — turning the Blacklist into a fair, guided
  progression with clear requirements and rewards.

**Continue:** [C54.5 — Reading GameFlow in RE](05-reading-gameflow.md) · [Chapter 54 hub](C54-GameFlow-Blacklist.md)
