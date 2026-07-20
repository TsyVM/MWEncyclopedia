# C61.4 — Traffic Behaviour

> **The one-sentence version:** each civilian car is an `AIVehicleTraffic` (195 methods) on `AIGoalTraffic` — a
> cheap brain that lane-follows the road network, swerves to avoid obstacles, and runs a crash-state machine when
> hit — a real `RBVehicle` you can wreck and use as a pursuit weapon.

[← C61.3 — Spawning](03-spawning.md) · [Chapter 61 hub](C61-Traffic-Ambient.md) ·
[Next: C61.5 — Reading traffic in RE →](05-reading-traffic.md)

---

## The civilian brain

Each traffic car is driven by an **`AIVehicleTraffic`** brain
([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md), 195 methods) running the **`AIGoalTraffic`** goal
([C46.2](../C46-AI-Goals-Actions/02-goal-catalog.md)) — the cheapest driver ([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md)),
because a civilian just needs to drive believably, not race or chase. Its behaviour is minimal:

- **Lane-follow** — drive along the road network's lanes ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md))
  at `TrafficSpeed`, obeying the road's flow.
- **Swerve-avoid** — dodge obstacles ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md), via
  `AvoidableManager`) — other cars, the player, hazards.
- **Crash-react** — when hit ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)), run a
  crash-state machine (spin, recover, or wreck).

So `AIVehicleTraffic`'s `AIGoalTraffic` menu ([C46.3](../C46-AI-Goals-Actions/03-data-only-goals.md)) is just
`Traffic` (lane-follow) + `TooDamaged` (crash) — the minimal repertoire of a civilian
([C46.2](../C46-AI-Goals-Actions/02-goal-catalog.md)). It's a *real* driver brain (195 methods, an `AIVehicle`
subclass) but running the simplest goal.

> ✅ *Verified:* `AIVehicleTraffic` (vtable `0x00891C08`, 195 methods, [C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md))
> runs `AIGoalTraffic` ([C46.2](../C46-AI-Goals-Actions/02-goal-catalog.md)); `TrafficCar`, `TrafficSpeed`, and the
> traffic sounds (`TrafficEngine`/`TrafficHorn`/`TrafficSkids`) are present.

## Traffic is a real physics car

Crucially, traffic is *not* a decoration — each is a **real `RBVehicle`**
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) with full physics
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)):

- **You can hit it** — a collision ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) with a
  traffic car is a real physics contact (`carhitcar`, [C43.3](../C43-Collision-Contacts/03-classification.md)) — it
  shoves you, damages you ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)), and wrecks *it*.
- **It reacts physically** — a hit traffic car spins, flips, tumbles — real rigid-body dynamics
  ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)), not a scripted animation.
- **It becomes an obstacle** — a wrecked traffic car sits in the road, a physical hazard for you *and* the cops.

So traffic is a *first-class physics participant* — the same `RBVehicle` machinery
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) as the player and cops, just driven cheaply. This
is why hitting traffic *matters*: it's a real collision with real consequences, not a harmless prop. The cost of
this (full physics per traffic car) is bounded by the moving window ([C61.2](02-traffic-density.md)) — only the
nearby cars are simulated — so the game affords *real* traffic without simulating a whole city's physics.

## Traffic as a pursuit weapon

Because traffic is real physics ([above](#traffic-is-a-real-physics-car)), it becomes a *tactical element* of the
pursuit ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)):

- **Cover** — weaving through dense traffic ([C61.2](02-traffic-density.md)) makes you *hard to follow* — cops must
  dodge the same cars, and can lose you in the flow (breaking line-of-sight for the bust envelope,
  [C48.4](../C48-Pursuit-Heat/04-bust-evade.md)).
- **A weapon** — spinning a traffic car into a cop's path ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md))
  can *disable or delay* the cop ([C45.5](../C45-Damage-Deformation/05-cop-damage.md)) — traffic as an improvised
  roadblock.
- **A hazard** — but traffic is also *your* danger — a mistimed weave means a crash that *slows you* (helping the
  cops, [C48.4](../C48-Pursuit-Heat/04-bust-evade.md)) and damages you ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)).

So traffic is a *double-edged* pursuit element — cover and weapon *and* hazard. This emerges *for free* from
traffic being real physics ([above](#traffic-is-a-real-physics-car)) — no special "traffic-as-weapon" code, just
the physics of hitting cars ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) applied to civilian
cars. It's the composition economy again ([C61.3](03-spawning.md)): traffic's tactical role is the collision system
+ real traffic physics, not a bespoke mechanic. This is why MW's pursuits through traffic feel so *alive* — the
river of cars is a real, physical part of the chase.

> 🟡 *Reasoned:* the traffic-as-cover/weapon/hazard tactics emerge from the verified real-`RBVehicle` traffic and
> the collision system ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)); the exact crash-state
> machine and avoidance tuning are deeper RE. The `AIVehicleTraffic` brain and its physics nature are verified.

## RE implications

- **`AIVehicleTraffic`** (195 methods) on `AIGoalTraffic` — a cheap brain: lane-follow, swerve-avoid, crash-react.
- **Traffic is a real `RBVehicle`** — full physics; you can hit, damage, and wreck it (real collisions, not props).
- **A pursuit element** — cover (weave to lose cops), weapon (spin into cops), hazard (a crash slows you).
- **Emerges from composition** — the tactical role is the collision system + real traffic physics, no bespoke code.

---

### Key takeaways

- Each civilian car is an **`AIVehicleTraffic`** (195 methods) on **`AIGoalTraffic`** — the cheapest driver:
  **lane-follow**, **swerve-avoid**, **crash-react** (its menu is just Traffic + TooDamaged).
- Traffic is a **real `RBVehicle`** with full physics — you can **hit, damage, and wreck** it (real `carhitcar`
  collisions with real consequences), not a harmless prop.
- It's a **double-edged pursuit element** — **cover** (weave to break cop line-of-sight), **weapon** (spin a car
  into a cop to disable it), and **hazard** (a mistimed weave crashes and slows *you*).
- Its tactical role **emerges from composition** — the collision system + real traffic physics — no bespoke
  "traffic weapon" code.
- The full-physics cost is **bounded by the moving window** ([C61.2](02-traffic-density.md)) — real traffic without
  simulating the whole city — which is why pursuits through traffic feel **alive**.

**Continue:** [C61.5 — Reading traffic in RE](05-reading-traffic.md) · [Chapter 61 hub](C61-Traffic-Ambient.md)
