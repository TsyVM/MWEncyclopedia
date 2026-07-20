# C54.2 — The CareerManager

> **The one-sentence version:** `CareerManager` owns the single-player campaign — the `CareerEvent`s you take on,
> the `CareerMode` progression state, the `CareerIntro` story beats, and the `CareerCrib` safehouse where you
> manage your cars.

[← C54.1 — The GameFlow state machine](01-gameflow-states.md) · [Chapter 54 hub](C54-GameFlow-Blacklist.md) ·
[Next: C54.3 — The Blacklist →](03-the-blacklist.md)

---

## The career authority

**`CareerManager`** is the owner of the *career* — Most Wanted's single-player campaign. Where `GameFlowStates`
([C54.1](01-gameflow-states.md)) is *what phase the game is in*, `CareerManager` is *where you are in the story*: it
holds your progression, unlocks, and the state of your climb up the Blacklist ([C54.3](03-the-blacklist.md)). Its
verified components:

- **`CareerMode`** — the campaign state (as opposed to quick-race or challenge modes).
- **`CareerEvent`** — an event in the career (a race, a challenge you take on for progression).
- **`CareerGame`** — the overall career-game state.
- **`CareerIntro`** — the story intros (the Blacklist rival cutscenes, staged by the director,
  [C53.3](../C53-Cameras-Director/03-cinematic-director.md)).
- **`CareerCrib`** — the safehouse/garage where you manage and customise your cars
  ([Chapter 56](../C56-Customization/C56-Customization.md)).

So `CareerManager` is the progression authority — it knows what you've completed, what's unlocked, which Blacklist
rival is next, and your bounty/milestone state ([C54.4](04-bounty-milestones.md)).

> ✅ *Verified:* `CareerManager`, `CareerMode`, `CareerEvent`, `CareerGame`, `CareerIntro`, and `CareerCrib` are
> present in `speed.exe` — the career system's components.

## CareerEvents: the units of progression

The career is made of **`CareerEvent`s** — the individual races and challenges that advance you
([Chapter 55](../C55-Race-Events/C55-Race-Events.md)):

- **Each event is a unit** — a specific race (circuit, sprint, [Chapter 55](../C55-Race-Events/C55-Race-Events.md))
  at a specific place, with a reward (bounty, unlocks, [C54.4](04-bounty-milestones.md)).
- **Events are gated** — some require prior events or a bounty threshold; completing events unlocks more and builds
  toward a Blacklist challenge ([C54.3](03-the-blacklist.md)).
- **Events feed the milestones** — winning events progresses your milestones
  ([C54.4](04-bounty-milestones.md)) toward the next rival.

So a `CareerEvent` is the *atom* of career progression — you take on events, win them, earn rewards, and advance.
`CareerManager` tracks which events you've done and which are available, presenting them in the front-end
([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) and loading them via `GameFlowLoadTrack`
([C54.1](01-gameflow-states.md)). The career is thus a *graph of events* with unlock gates, and `CareerManager` is
your position in it.

## The CareerCrib: home base

**`CareerCrib`** is the *safehouse* — your home base between events. It's where the career loop *rests*:

- **Manage cars** — view your garage, switch cars, and customise
  ([Chapter 56](../C56-Customization/C56-Customization.md)) — the crib is the gateway to the customization system.
- **Check progress** — the Blacklist ([C54.3](03-the-blacklist.md)), your bounty and milestones
  ([C54.4](04-bounty-milestones.md)), what's next.
- **Launch events** — from the crib/map, choose the next `CareerEvent` to take on.

So the crib is the *hub* of the career front-end ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) —
the place you return to between races to tune your car and pick your next challenge. In the game's rhythm
([C54.1](01-gameflow-states.md)), the crib is the "front-end" pole of the front-end↔track loop, specialised for the
career: it's where you *prepare* (customise, choose) before *playing* (the event). This gives the career a sense of
*home* — a base you build up (better cars) and return to, between the escalating challenges of the Blacklist.

> 🟡 *Reasoned:* the CareerCrib's role as the customization/progression hub is inferred from its name and the
> career/customization structure; the exact crib UI and functions are per-screen RE. The career classes are
> verified.

## Why a career manager

Centralising the campaign in a `CareerManager` (rather than scattering progression state) is the same
single-authority design as `GameFlowStates` ([C54.1](01-gameflow-states.md)):

- **One source of progression truth.** `CareerManager` holds *the* career state — completed events, unlocks,
  Blacklist position, bounty. Every system that needs it (the front-end, the save,
  [Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)) asks it.
- **The save is the career state.** Saving the game ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)) is
  essentially serialising `CareerManager`'s state — your progression *is* the save.
- **It drives the front-end.** The front-end ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md))
  presents what `CareerManager` says is available — the Blacklist, the events, the crib.

So `CareerManager` is the *spine* of the single-player game — the progression authority that the front-end
displays, the save persists, and the Blacklist climb ([C54.3](03-the-blacklist.md)) advances. It turns a collection
of events and pursuits into a *campaign* with a beginning (Blacklist #15), a middle (the climb), and an end (#1).
Understanding the career is understanding `CareerManager`: the object that *is* your journey through Most Wanted.

## RE implications

- **`CareerManager`** owns the campaign — `CareerEvent`s, `CareerMode`, `CareerIntro`, `CareerCrib`.
- **`CareerEvent`s** are the units of progression — gated races/challenges that build toward Blacklist challenges.
- **`CareerCrib`** is the home base — manage cars, check progress, launch events (the career front-end hub).
- **A career manager** centralises progression — one truth, the save's content, the front-end's driver.

---

### Key takeaways

- **`CareerManager`** owns the single-player **career** — the progression authority holding your completed events,
  unlocks, Blacklist position, and bounty/milestone state.
- The career is built of **`CareerEvent`s** — gated races/challenges (the units of progression) that feed the
  milestones toward the next Blacklist rival.
- The **`CareerCrib`** (safehouse) is the **home base** — manage/customise cars, check progress, launch events —
  the career front-end hub.
- A **centralised career manager** is the single source of progression truth — **the save's content** and the
  front-end's driver.
- `CareerManager` turns events and pursuits into a **campaign** with an arc from Blacklist #15 to #1.

**Continue:** [C54.3 — The Blacklist](03-the-blacklist.md) · [Chapter 54 hub](C54-GameFlow-Blacklist.md)
