# Chapter 54 — GameFlow, Modes & the Blacklist

> **Goal of this chapter:** decode the career structure that strings the game together — the top-level
> `GameFlowStates` machine (front-end ↔ track ↔ cutscene), the `CareerManager` and its `CareerEvent`s, the
> **Blacklist** of 15 rivals, and the **Bounty**/**Milestone** systems that gate progression up the list.

Individual races ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) and pursuits
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) are the game's *moments*; this chapter is the *structure*
that connects them — the career. It decodes the `GameFlowStates` machine that moves you between the front-end and
the track, the `CareerManager` that owns your progression, the Blacklist of Most Wanted rivals you climb, and the
Bounty and Milestone systems that decide when you can challenge the next one. This is the *shape* of Most Wanted:
the loop from race, to bounty, to Blacklist challenge, to the next rival.

> **Verified against the executable.** The top-level state machine is **`GameFlowStates`**, with transitions like
> `GameFlowLoadTrack`, `GameFlowLoadingFrontEnd`, `GameFlowLoadingFrontEndPart`. The career is **`CareerManager`**
> owning `CareerMode`, `CareerEvent`, `CareerGame`, `CareerIntro`, and `CareerCrib` (the safehouse). The
> **`Blacklist`** is named. Progression is gated by **`Milestone*`** (`MilestoneAtStake`, `MilestoneProgress`,
> `MilestoneReached`, `MilestoneRewards`, `MilestonesScreen`, `milestonejump`) and **`Bounty`** (`BountyDatum`).
> The front-end is `FrontEnd`/`FrontEndPart`.

---

## Deep-dive pages

- [C54.1 — The GameFlow state machine](01-gameflow-states.md): the top-level phases and transitions.
- [C54.2 — The CareerManager](02-career-manager.md): career mode, events, and the crib.
- [C54.3 — The Blacklist](03-the-blacklist.md): the 15 rivals and the climb.
- [C54.4 — Bounty & Milestones](04-bounty-milestones.md): the gates on progression.
- [C54.5 — Reading GameFlow in RE](05-reading-gameflow.md): navigating the career structure.

---

## 54.1 The GameFlow state machine

**`GameFlowStates`** ([C54.1](01-gameflow-states.md)) is the game's *top-level state machine* — the phases the whole
game moves between: the front-end (menus, `GameFlowLoadingFrontEnd`), loading a track (`GameFlowLoadTrack`),
in-game (racing/roaming), and cutscenes. It's the master `GameFlow` referenced by the resource-streaming phases
([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)) — each state has its
resident set, and transitions ([Chapter 38](../C38-Resource-Streaming-Residency/04-gameflow.md)) load the next
state's resources (behind the loading screens, [C38.6](../C38-Resource-Streaming-Residency/06-blocking-budgets.md)).

## 54.2 The CareerManager

**`CareerManager`** ([C54.2](02-career-manager.md)) owns the *career* — the single-player campaign. It manages
`CareerEvent`s (the races and challenges you take on), `CareerMode` (the campaign state), `CareerIntro` (the story
intros, staged by the cinematic director, [Chapter 53](../C53-Cameras-Director/03-cinematic-director.md)), and
`CareerCrib` (the safehouse where you manage your cars, [Chapter 56](../C56-Customization/C56-Customization.md)).
It's the progression authority — what you've done, what's unlocked, where you are.

## 54.3 The Blacklist

The **`Blacklist`** ([C54.3](03-the-blacklist.md)) is Most Wanted's defining structure — the **15 rivals** you
climb from #15 to #1, each a boss you must out-earn and defeat. The Blacklist *is* the career's spine: you race
events to build bounty ([C54.4](04-bounty-milestones.md)), meet the requirements to challenge the next rival, beat
them, and take their spot — the ladder that gives the game its name and its arc from unknown to Most Wanted.

## 54.4 Bounty & Milestones

Progression up the Blacklist is gated by **Bounty** and **Milestones** ([C54.4](04-bounty-milestones.md)):
**Bounty** (`BountyDatum`) is the reward you earn from pursuits and events — your notoriety, the currency of the
Blacklist. **Milestones** (`MilestoneAtStake`, `MilestoneProgress`, `MilestoneRewards`) are the specific
requirements to challenge a rival — races to win, a bounty threshold, pursuit objectives. Meeting a rival's
milestones ([C54.4](04-bounty-milestones.md)) unlocks the challenge; the `MilestonesScreen` shows your progress.

---

### Key takeaways

- **`GameFlowStates`** is the game's **top-level state machine** — front-end ↔ track ↔ cutscene — driving the
  resource-streaming phases ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
- **`CareerManager`** owns the career — `CareerEvent`s, `CareerMode`, `CareerIntro`, and the `CareerCrib`
  safehouse.
- The **`Blacklist`** — **15 rivals** climbed from #15 to #1 — is the career's spine and the game's namesake arc.
- Progression is gated by **Bounty** (`BountyDatum` — earned notoriety) and **Milestones** (`MilestoneAtStake`/
  `Rewards` — the requirements to challenge the next rival).
- The loop is **race → earn bounty → meet milestones → challenge the Blacklist rival → take their spot → repeat**.

**Next:** [Chapter 55 — Race Events & Game Modes](../C55-Race-Events/C55-Race-Events.md): the events that fill the
career.
