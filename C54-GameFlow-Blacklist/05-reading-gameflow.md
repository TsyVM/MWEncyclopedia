# C54.5 — Reading GameFlow in RE

> **The one-sentence version:** navigate the career by `GameFlowStates` (the top-level machine), `CareerManager`
> (progression), the `Blacklist` (the ladder), and the `Bounty`/`Milestone*` systems (the gates) — reading the
> structure that turns the game's moments into a campaign.

[← C54.4 — Bounty & Milestones](04-bounty-milestones.md) · [Chapter 54 hub](C54-GameFlow-Blacklist.md) ·
[Next: Chapter 55 — Race Events & Game Modes →](../C55-Race-Events/C55-Race-Events.md)

---

## Anchors for career RE

The career structure is anchored on verified classes:

- **`GameFlowStates`** — the top-level state machine ([C54.1](01-gameflow-states.md)).
- **`CareerManager`** and its parts — `CareerEvent`, `CareerMode`, `CareerCrib` ([C54.2](02-career-manager.md)).
- **The `Blacklist`** — the rival ladder ([C54.3](03-the-blacklist.md)).
- **The `Bounty`/`Milestone*` systems** — the progression gates ([C54.4](04-bounty-milestones.md)).

From these, the career is navigable: the phases, the progression, the ladder, and the gates.

## The RE workflow

Reading the career:

1. **Trace `GameFlowStates`** — the top-level phases and transitions ([C54.1](01-gameflow-states.md)); how the game
   moves front-end ↔ track.
2. **Map the `CareerManager`** — the events, mode, and crib ([C54.2](02-career-manager.md)); the progression
   authority.
3. **Read the `Blacklist`** — the 15-rung ladder ([C54.3](03-the-blacklist.md)) and its rival gates.
4. **Follow the gates** — bounty and milestones ([C54.4](04-bounty-milestones.md)) that unlock each rung.

The output is the full career picture: phases, progression, ladder, and gates.

## The career is the save

A key RE insight: **the career state is essentially the save game** ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)).
`CareerManager`'s state ([C54.2](02-career-manager.md)) — your completed events, unlocks, Blacklist position, bounty,
and milestone progress — *is* what the save persists. So:

- **Reading a save** ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)) is reading the `CareerManager`
  state — where you are in the campaign.
- **The career classes define the save schema** — `CareerEvent` completion, `MilestoneReached`, `BountyDatum` are
  the persisted fields.
- **Progression is durable** — it survives across sessions because it's saved, unlike the transient pursuit/render
  state.

So the career and the save are two views of one thing — the progression, held live by `CareerManager` and persisted
by the save. This ties the top-level structure ([C54.1](01-gameflow-states.md)) to the save system
([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)): what you *are* in the game (your rank, cars,
progress) is the career state, saved. Reading the career thus doubly rewards — it decodes both the campaign
structure and the save format.

## The career ties the game together

With the career decoded, the *shape* of Most Wanted is clear — it's the frame that holds every other system:

- **The events** ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) are the `CareerEvent`s you play.
- **The pursuits** ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) earn the bounty
  ([C54.4](04-bounty-milestones.md)) that advances you.
- **The cars** ([Chapter 56](../C56-Customization/C56-Customization.md)) are won from Blacklist rivals
  ([C54.3](03-the-blacklist.md)) and tuned in the crib ([C54.2](02-career-manager.md)).
- **The world** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) is loaded per `GameFlowStates`
  ([C54.1](01-gameflow-states.md)).

So the career is the *organising structure* — the reason to race, to pursue, to customise: the climb up the
Blacklist. Every system serves it: you race and pursue to earn bounty and milestones, to challenge rivals, to win
cars, to climb to #1. Reading the career last (of the game-structure chapters) shows how the pieces cohere into a
*game* — not just a set of systems, but a campaign with a purpose. It's the top of the pyramid the whole book has
been building toward: the machinery ([Part VII](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)), the
content ([Part VIII](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), and the presentation
([Part IX](../C51-Render-Pipeline/C51-Render-Pipeline.md)), all in service of the climb.

## RE implications

- **Anchor on** `GameFlowStates`, `CareerManager`, the `Blacklist`, and the `Bounty`/`Milestone*` systems.
- **The RE workflow** — trace the state machine → map the career → read the Blacklist → follow the gates.
- **The career is the save** — `CareerManager`'s state is what's persisted
  ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)).
- **The career ties the game together** — every system serves the climb up the Blacklist.

---

### Key takeaways

- The career is anchored on **`GameFlowStates`** (top-level machine), **`CareerManager`** (progression), the
  **`Blacklist`** (ladder), and the **`Bounty`/`Milestone*`** systems (gates).
- The RE workflow: **trace `GameFlowStates` → map `CareerManager` → read the `Blacklist` → follow the bounty/
  milestone gates**.
- **The career state is the save** — `CareerManager`'s completed events, unlocks, Blacklist rank, bounty, and
  milestones are what the save persists ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)).
- The career **ties the whole game together** — the events, pursuits, cars, and world all serve the **climb up the
  Blacklist** to #1.
- Reading the career shows how the machinery, content, and presentation **cohere into a campaign** with a purpose.

**Next:** [Chapter 55 — Race Events & Game Modes](../C55-Race-Events/C55-Race-Events.md): the events that fill the
career.

**Sources:** `speed.exe` (verified: `GameFlowStates`/`GameFlowLoadTrack`/`GameFlowLoadingFrontEnd`; `CareerManager`/
`CareerMode`/`CareerEvent`/`CareerGame`/`CareerIntro`/`CareerCrib`; `Blacklist`; `Milestone`/`MilestoneAtStake`/
`MilestoneProgress`/`MilestoneReached`/`MilestoneRewards`/`MilestonesScreen`/`MilestoneBoard`/`MilestoneDatum`;
`Bounty`/`BountyDatum`; `FrontEnd`/`FrontEndPart`).
