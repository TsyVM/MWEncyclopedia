# C61.2 — Traffic Density

> **The one-sentence version:** `TrafficLevel` sets how much traffic the manager maintains, and traffic lives in a
> ring around the camera — spawned ahead out of view, populating the roads near you, despawned behind — a moving
> window that keeps the world populated without simulating the whole city.

[← C61.1 — The traffic system](01-traffic-system.md) · [Chapter 61 hub](C61-Traffic-Ambient.md) ·
[Next: C61.3 — Spawning →](03-spawning.md)

---

## TrafficLevel: the density dial

**`TrafficLevel`** (verified) is the *density* setting — how many civilian cars the manager
([C61.1](01-traffic-system.md)) maintains per area. It ranges from empty to busy:

- **Low** — sparse roads, the occasional car — quieter, easier to drive fast.
- **High** — a busy city, lots of traffic — harder to weave, more obstacles.

`TrafficLevel` can be set globally (a game/difficulty option) or vary by *zone* — downtown is busier than the
outskirts ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)). It's the dial the manager works to
maintain: if the current density is below `TrafficLevel`, spawn more ([C61.3](03-spawning.md)); if above, let it
thin. So the density is a *target* the manager continuously matches — the same maintain-a-target logic as the cop
count ([C49.3](../C49-Cops-Dispatch-Roadblocks/03-formations-dispatch.md)).

> ✅ *Verified:* `TrafficLevel` and `TrafficSpeed` are present in `speed.exe` — the density and ambient-speed
> settings. `AITrafficManager` maintains the population to the level.

## The ring around the camera

Traffic exists in a **ring (or window) around the camera** ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md))
— the moving region where cars are spawned and simulated:

```
        despawn (behind, out of view)
              ↑
   [ ===== active traffic near the player ===== ]
              ↓
        spawn (ahead, out of view)
```

- **Spawn ahead** — cars appear *out of view* ahead of you (over a rise, around a corner), so you never see them
  pop in.
- **Active near you** — the cars on the roads around you are live, drivable-into
  ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)), avoidable
  ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)).
- **Despawn behind** — cars you've passed, now *out of view* behind, are removed (their slots freed for new spawns
  ahead).

So the traffic is a *moving window* that follows the player — always populated around you, never simulating the
*whole* city. As you drive, the window moves: new cars spawn ahead, old ones despawn behind, maintaining the
density ([above](#trafficlevel-the-density-dial)) in your vicinity. This is the same principle as world streaming
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — keep resident only what's near the player — applied
to *dynamic actors* rather than static geometry.

> 🟡 *Reasoned:* the spawn-ahead/despawn-behind ring around the camera is the standard open-world traffic model,
> consistent with the verified `AITrafficManager`/`TrafficLevel` and the streaming philosophy
> ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)); the exact ring geometry is per-config. The
> manager and density setting are verified.

## Why a moving window

Maintaining traffic in a *moving window* (rather than simulating the whole city's cars) is essential for
performance ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)):

- **Bounded cost.** Only the ~dozens of cars near the player are simulated
  ([C61.4](04-traffic-behavior.md)) — not the thousands a whole city would have. The traffic cost is *constant*
  regardless of city size.
- **Always populated.** Because the window follows you, there's *always* traffic around you — the world never
  looks empty where you are. The illusion of a full city is maintained by populating only your vicinity.
- **Reuses slots.** Despawned cars ([above](#the-ring-around-the-camera)) free their pool slots
  ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) for new spawns — so the population *churns*
  through a fixed pool, bounded memory.

So the moving window is the traffic system's core trick: a *local* simulation that *appears* global. You see a busy
city, but only your neighbourhood is real; the rest is spawned as you approach and forgotten as you leave. This is
the same *pay-for-what's-near* economy as streaming ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)),
smackables ([C43.5](../C43-Collision-Contacts/05-smackables.md)), and the cull tree
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) — the recurring open-world principle that makes a vast
world affordable: simulate the window, not the world.

## RE implications

- **`TrafficLevel`** is the density dial — the target population the manager maintains (globally or per zone).
- **Traffic lives in a ring around the camera** — spawned ahead (out of view), active near you, despawned behind.
- **A moving window** — always populated around the player, without simulating the whole city.
- **Pay-for-what's-near** — bounded cost, always-populated illusion, churning a fixed pool — the open-world
  principle.

---

### Key takeaways

- **`TrafficLevel`** sets the **density** — the target civilian population the manager maintains (from sparse to
  busy, globally or per zone) — the same maintain-a-target logic as the cop count.
- Traffic lives in a **ring around the camera** — **spawned ahead** (out of view), **active near** you (drivable-into,
  avoidable), **despawned behind** — so cars never visibly pop in or out.
- It's a **moving window** that follows the player — **always populated** around you, **never simulating the whole
  city**.
- The window gives **bounded cost** (only ~dozens of cars simulated), an **always-full illusion**, and a **churning
  fixed pool** (despawns free slots).
- This is the **pay-for-what's-near** open-world principle — simulate the *window*, not the *world* — shared with
  streaming, smackables, and culling.

**Continue:** [C61.3 — Spawning](03-spawning.md) · [Chapter 61 hub](C61-Traffic-Ambient.md)
