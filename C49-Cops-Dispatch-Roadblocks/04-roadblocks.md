# C49.4 — Roadblocks

> **The one-sentence version:** `AIRoadBlock` (62 methods) runs a verified `Roadblock_*` lifecycle — `CallForRB`
> (AIPursuit requests one) → site selection ahead of you → `RBApproach` → `RBEngage` (cruisers parked across the
> road on `AIGoalStaticRoadBlock`) → `RBAverted`/dodged — with `roadblocks_dodged` tracking your evasions.

[← C49.3 — Formations & dispatch](03-formations-dispatch.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md) ·
[Next: C49.5 — Spikes & pursuit breakers →](05-spikes-breakers.md)

---

## The roadblock lifecycle

A **roadblock** is a set-piece: cruisers parked across the road ahead to stop or slow you. It's orchestrated by
**`AIRoadBlock`** ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md)) through a **verified `Roadblock_*`
lifecycle** — a sequence of named events recovered from `speed.exe`:

```
Roadblock_CallForRB          AIPursuit requests a roadblock (high Heat, C48)
   → (site selection)         AIRoadBlock picks a spot ahead of you on the road net
   → Roadblock_RBApproach     cruisers move to the site and park
   → Roadblock_RBEngage       the block is live — cars across the road, ready
   → Roadblock_RBAverted      you got past (dodged) — the block is torn down
   → Roadblock_RBReminder     (chatter/warning cues during approach)
```

Plus the dispatch/reply events (`Roadblock_DispRBReply`, `Roadblock_DispSubRB`, `Roadblock_NegativeRBReply`) — the
coordination chatter between the pursuit director and the roadblock orchestrator. This lifecycle is the roadblock's
state machine, verified event by event.

> ✅ *Verified:* the roadblock lifecycle events are present in `speed.exe` — `Roadblock_CallForRB`,
> `Roadblock_RBApproach`, `Roadblock_RBEngage`, `Roadblock_RBAverted`, `Roadblock_RBReminder`,
> `Roadblock_DispRBReply`, `Roadblock_DispSubRB`, `Roadblock_NegativeRBReply`, plus the stats
> `roadblocks_dodged`/`roadblocks_dodged_in_pursuit`. `AIRoadBlock` is vtable `0x00892130`, 62 methods.

## CallForRB: the request

The lifecycle begins with **`Roadblock_CallForRB`** — `AIPursuit` ([Chapter 48](../C48-Pursuit-Heat/01-the-cast.md))
requests a roadblock when Heat authorises it ([Chapter 48](../C48-Pursuit-Heat/03-escalation-ladder.md),
`PursuitAddsRoadblock`). This isn't unconditional — the request goes through a dispatch handshake
(`Roadblock_DispRBReply` / `Roadblock_NegativeRBReply`): the fleet may *decline* (no cruisers available, no valid
site), replying negatively. So a called-for roadblock isn't guaranteed — the fleet has to *have the units and a
place to put them*. This handshake is why roadblocks appear at plausible moments (a straight ahead, enough cops
free) rather than materialising anywhere.

## Site selection and RBApproach

If the request is accepted, `AIRoadBlock` **selects a site** — a spot **ahead of you on the road network**
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)). This uses the same road graph the AI navigates
([Chapter 47](../C47-AI-Driver-Vehicle/04-navigation-systems.md)): find a road segment far enough ahead (so cops
can reach it before you), wide enough to block, on your likely path. Then **`Roadblock_RBApproach`**: the reserved
cruisers drive to the site and park in formation across the road, installing `AIGoalStaticRoadBlock` /
`AIActionStaticRoadBlock` ([Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md)) — the "hold this position"
intent. That `AIGoalStaticRoadBlock` is vault-tuned (×11, [Chapter 46](../C46-AI-Goals-Actions/02-goal-catalog.md))
reflects the parameters of a good block — spacing, angle, how many cars.

That `AIRoadBlock` has **62 methods for a 168-byte object** ([Chapter 47](../C47-AI-Driver-Vehicle/03-managers.md))
confirms its nature: most of its code is *siting and formation* (the geometry of where and how to place the block),
with little persistent state — a computation-heavy, memory-light orchestrator.

## RBEngage and RBAverted: the outcome

Once parked, **`Roadblock_RBEngage`** — the block is *live*: cars across the road, ready for your arrival. Then one
of two outcomes:

- **You hit it** — a collision ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) with the parked
  cruisers, potentially wrecking you or scrubbing your speed (feeding the bust envelope,
  [Chapter 48](../C48-Pursuit-Heat/04-bust-evade.md)).
- **You dodge it** — **`Roadblock_RBAverted`**: you got past (through a gap, around, or over). The block is torn
  down, and the stat `roadblocks_dodged` (`roadblocks_dodged_in_pursuit`) increments — the game *tracks* your
  evasions ([Chapter 48](../C48-Pursuit-Heat/04-bust-evade.md), for pursuit rewards/records).

So a roadblock is a *timed threat* on your path — reach it and either crash or thread it. That the game counts
dodged roadblocks (a verified stat) shows they're a scored skill moment, not just an obstacle. The whole lifecycle
— call, site, approach, engage, avert — is the anatomy of that set-piece.

## Why a lifecycle with a handshake

Building roadblocks as a full lifecycle with a request/reply handshake ([above](#callforrb-the-request)) is
deliberate:

- **Roadblocks must be *earned* by the situation** — enough Heat to authorise, enough cops free, a valid site
  ahead. The handshake enforces this, so roadblocks feel *deployed*, not spawned arbitrarily.
- **They take *time* to set up** — call → approach → engage is not instant, giving you warning (chatter,
  `RBReminder`) and a chance to reroute. This is fair: a roadblock you couldn't possibly avoid would feel cheap.
- **They *resolve* cleanly** — hit or averted, then torn down (`RBAverted`), returning the cruisers to the chase.
  No stale blocks litter the map.

So the roadblock lifecycle is a small, complete drama — requested, built, faced, resolved — that adds a tactical
beat to a high-Heat pursuit. It's `AIRoadBlock` conducting a subset of the fleet through a scripted set-piece, all
verified event by event ([C49.6](06-reading-fleet.md)).

## RE implications

- **`AIRoadBlock` runs a verified `Roadblock_*` lifecycle** — CallForRB → site → RBApproach → RBEngage →
  RBAverted.
- **A request/reply handshake** (`DispRBReply`/`NegativeRBReply`) means roadblocks are *earned*, not guaranteed.
- **Cruisers hold via `AIGoalStaticRoadBlock`** (vault-tuned ×11) — parked in formation across the road.
- **Dodged roadblocks are scored** — `roadblocks_dodged` is a tracked stat.

---

### Key takeaways

- Roadblocks are orchestrated by **`AIRoadBlock`** through a **verified `Roadblock_*` lifecycle**: `CallForRB` →
  site selection → `RBApproach` → `RBEngage` → `RBAverted`/dodged.
- The request goes through a **handshake** (`DispRBReply`/`NegativeRBReply`) — a roadblock is *earned* (enough Heat,
  free cops, a valid site), not guaranteed.
- Cruisers park in formation on **`AIGoalStaticRoadBlock`** (vault-tuned, ×11); `AIRoadBlock`'s 62 methods are
  mostly **siting and formation** geometry.
- The outcome is **hit or dodge** — dodging increments the verified stat `roadblocks_dodged` (a scored skill
  moment).
- The lifecycle makes roadblocks a **fair, timed set-piece** — announced, avoidable, and cleanly resolved.

**Continue:** [C49.5 — Spikes & pursuit breakers](05-spikes-breakers.md) · [Chapter 49 hub](C49-Cops-Dispatch-Roadblocks.md)
