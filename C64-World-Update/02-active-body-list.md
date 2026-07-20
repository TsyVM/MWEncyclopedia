# C64.2 — The Active-Body List

> **The one-sentence version:** `World_UpdateBody` walks the active-body list — every body being simulated (cars,
> dynamic props) — advancing each; bodies register when active and deregister when inert, so the list is always
> the current set of simulated things.

[← C64.1 — The world tick](01-world-tick.md) · [Chapter 64 hub](C64-World-Update.md) ·
[Next: C64.3 — One-shot effects →](03-one-shot-effects.md)

---

## Walking the bodies

**`World_UpdateBody`** ([C64.1](01-world-tick.md)) is the sub-update that advances the *bodies* — it walks the
**active-body list** and ticks each:

```
World_UpdateBody:
   for each body on the active-body list:
      advance it — its Physics::Simulate (C39.2), integrate (C39.4)
```

So the active-body list is the *set of bodies being simulated* this frame — the player's car, the cops, the
traffic ([Chapter 61](../C61-Traffic-Ambient/C61-Traffic-Ambient.md)), any active dynamic props. `World_UpdateBody`
is the loop that drives them: for each, run its per-body tick ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).
This is the *entry point* to the vehicle simulation ([C39.1](../C39-Vehicle-Simulation/01-pipeline.md)) — the sim
driver ([C39.1](../C39-Vehicle-Simulation/01-pipeline.md)) is reached from here, `World_UpdateBody` calling each
body's `Simulate`.

> ✅ *Verified:* `World_UpdateBody` and `WorldBodyConn` ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) are
> present in `speed.exe`; the bodies it walks are the `RigidBody` tree
> ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)).

## Register and deregister

The active-body list has *dynamic membership* — bodies join and leave as they activate and deactivate
([C64.1](01-world-tick.md)):

- **Register when active** — a body that needs simulating (a car, a knocked smackable,
  [C43.5](../C43-Collision-Contacts/05-smackables.md)) *registers* on the list, so it's ticked each frame.
- **Deregister when inert** — a body that no longer needs simulating (a settled smackable going back to sleep,
  [C43.5](../C43-Collision-Contacts/05-smackables.md); a despawned traffic car,
  [C61.2](../C61-Traffic-Ambient/02-traffic-density.md)) *deregisters*, so it stops costing update time.

So the list is *self-maintaining* — it always holds exactly the bodies that need simulating *now*. This is the
mechanism behind the sleep/wake economy ([C43.5](../C43-Collision-Contacts/05-smackables.md)): a smackable "sleeps"
by *deregistering* from the active-body list (so it's inert, costing nothing), and "wakes" by *registering* (so
it's simulated). The list *is* the boundary between active and inert. Reading it explains how thousands of
smackables ([C43.5](../C43-Collision-Contacts/05-smackables.md)) cost nothing until hit — they're not on the active
list until they wake.

> 🟡 *Reasoned:* the register/deregister-on-active/inert mechanism is the natural reading of the active-body list
> and the verified sleep/wake economy ([C43.5](../C43-Collision-Contacts/05-smackables.md)); the exact list
> implementation is deeper RE. `World_UpdateBody` and the bodies are verified.

## WorldBodyConn: the body's world link

Each active body connects to the world via **`WorldBodyConn`** ([C39.5](../C39-Vehicle-Simulation/05-connectors.md))
— the connector linking a body to the `World`:

- **Registration** — a body's `WorldBodyConn` is what puts it on (and takes it off) the active-body list.
- **World services** — through it, the body accesses world services (collision,
  [Chapter 63](../C63-Collision-World/C63-Collision-World.md); the section it's in,
  [Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).

So `WorldBodyConn` is the body's *membership card* in the world — how it's registered for updating and connected to
the world's systems. This is the connector pattern ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) applied to
world membership: a body doesn't manage its own world registration; its `WorldBodyConn` does, one-way
([C39.5](../C39-Vehicle-Simulation/05-connectors.md)). It's the link that makes a body *part of the world* — on the
active list, in a section, in the collision world — so `World_UpdateBody` can find and tick it.

## The active-body list is the sim's population

The active-body list ([above](#walking-the-bodies)) is, in effect, the *population of the simulation* — the set of
everything the physics ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) is advancing:

- **The cars** — player, cops ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)),
  traffic ([Chapter 61](../C61-Traffic-Ambient/C61-Traffic-Ambient.md)).
- **The active props** — smackables mid-scatter ([C43.5](../C43-Collision-Contacts/05-smackables.md)), debris
  ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)), the trailer
  ([Chapter 62](../C62-Constraints-Joints/C62-Constraints-Joints.md)).

So reading the active-body list ([C64.5](05-reading-world-update.md)) in a memory dump tells you *everything being
simulated right now* — the live physics population. It's the counterpart, for *dynamics*, of the streaming
manager's resident-section list ([C38.2](../C38-Resource-Streaming-Residency/02-sections-residency.md)) for
*geometry*: one lists what's *loaded*, the other what's *simulating*. Together they define the live game state —
what's in memory and what's moving. The active-body list is the *dynamic* half of the world's live state, walked by
`World_UpdateBody` each frame.

## RE implications

- **`World_UpdateBody`** walks the active-body list — advancing each body's tick
  ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).
- **Dynamic membership** — bodies register when active, deregister when inert (the sleep/wake economy).
- **`WorldBodyConn`** is the body's world link — its registration and world-service access.
- **The active-body list is the sim's population** — everything being simulated now; the dynamics half of the live
  state.

---

### Key takeaways

- **`World_UpdateBody`** walks the **active-body list** — every body being simulated (cars, dynamic props) —
  advancing each via its per-body tick ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)); it's
  the entry to the sim driver.
- The list has **dynamic membership** — bodies **register** when active, **deregister** when inert — the mechanism
  behind the **sleep/wake economy** (a slept smackable leaves the list, costing nothing).
- **`WorldBodyConn`** is the body's **world membership** — its registration and access to world services
  (collision, section).
- The active-body list is the **simulation's population** — everything being simulated *now*; reading it in a dump
  gives the live physics population.
- It's the **dynamic half** of the world's live state — the counterpart to the streaming manager's resident-section
  list (what's *loaded* vs. what's *simulating*).

**Continue:** [C64.3 — One-shot effects](03-one-shot-effects.md) · [Chapter 64 hub](C64-World-Update.md)
