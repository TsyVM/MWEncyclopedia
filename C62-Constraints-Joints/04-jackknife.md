# C62.4 — The Jackknife

> **The one-sentence version:** `JackKnife` (via `AIActionJackKnife`) turns a truck-and-trailer into a pursuit
> roadblock — a cop-directed semi swinging its trailer across the road to block your path — the articulated
> physics weaponised.

[← C62.3 — Trailers](03-trailers.md) · [Chapter 62 hub](C62-Constraints-Joints.md) ·
[Next: C62.5 — Reading constraints in RE →](05-reading-constraints.md)

---

## The jackknife maneuver

A **jackknife** is when a truck's trailer swings out to the side, folding at the hitch
([C62.2](02-joints-coupling.md)) until the truck-and-trailer forms an "L" or "V" across the road — a dramatic loss
of control (for a real truck) or a *deliberate blockade* (in a pursuit). Most Wanted has the verified **`JackKnife`**,
executed by the AI action **`AIActionJackKnife`** ([C46.5](../C46-AI-Goals-Actions/05-action-menu.md)):

- **A cop-directed truck** — during a pursuit ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), a truck can
  be directed (by the AI, [Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) to jackknife across your
  path.
- **The trailer swings across** — the articulated physics ([C62.3](03-trailers.md)) — the trailer pivots about the
  hitch until it spans the road, forming a rolling roadblock.
- **A blockade** — the jackknifed rig blocks the road ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
  — you must dodge, brake, or crash into it.

So the jackknife is the *weaponised* form of trailer physics — the articulation ([C62.3](03-trailers.md)) that
makes a trailer swing, used deliberately to block you. `AIActionJackKnife` ([C46.5](../C46-AI-Goals-Actions/05-action-menu.md))
is the AI action ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) that commands it.

> ✅ *Verified:* `JackKnife` is present in `speed.exe`; `AIActionJackKnife` ([C46.5](../C46-AI-Goals-Actions/05-action-menu.md))
> is a verified AI action. The trailer articulation ([C62.3](03-trailers.md)) is the mechanism it uses.

## A dynamic roadblock

The jackknife is a *different kind* of roadblock than the static one
([C49.4](../C49-Cops-Dispatch-Roadblocks/04-roadblocks.md)):

| | Static roadblock ([C49.4](../C49-Cops-Dispatch-Roadblocks/04-roadblocks.md)) | Jackknife (this page) |
|---|---|---|
| Made of | parked cruisers ([C49.4](../C49-Cops-Dispatch-Roadblocks/04-roadblocks.md)) | a jackknifing truck-and-trailer |
| Setup | pre-positioned ahead ([C49.4](../C49-Cops-Dispatch-Roadblocks/04-roadblocks.md)) | dynamic, in front of you |
| Physics | static bodies | articulated jackknife ([C62.3](03-trailers.md)) |
| Feel | a wall to thread | a sweeping, rolling block |

So the jackknife is a *dynamic, physics-driven* roadblock — a truck actively swinging its trailer across the road,
as opposed to cruisers parked in advance. This makes it more *dramatic* and *unpredictable* — the jackknife happens
*in motion*, the trailer sweeping across, so you must react to a *moving* blockade. It's the pursuit's most
spectacular obstacle, made possible by the articulated trailer physics ([C62.3](03-trailers.md)). The engine's
investment in trailer stabilisation (`SuspensionTrailer`, 99 methods,
[C62.3](03-trailers.md)) pays off here: a *controlled* jackknife (swinging the trailer *just* across the road, not
flipping the truck) requires exactly that stabilisation code.

> 🟡 *Reasoned:* the jackknife-as-dynamic-roadblock role is the natural reading of the verified `JackKnife`/
> `AIActionJackKnife` and the pursuit roadblock system ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md));
> the exact jackknife-trigger and control are deeper RE. The `JackKnife` string and `AIActionJackKnife` are
> verified.

## Physics as spectacle

The jackknife exemplifies a theme: Most Wanted turns its *physics* into *spectacle*. The articulated trailer
([C62.3](03-trailers.md)) — a hard simulation problem — isn't just for realism; it's *the enabling technology* for
a dramatic set-piece (the jackknife). This is the engine's physics investment paying off in *drama*:

- **The hard case is the cool case** — the trailer (hardest to simulate, [C62.3](03-trailers.md)) enables the
  jackknife (most spectacular obstacle). The difficulty *is* the payoff.
- **Real physics, real drama** — because the jackknife is *actual articulated physics*
  ([C62.1](01-constraints.md)), not a scripted animation, it interacts truly with you (you can clip it, dodge it,
  crash into it, [Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) — a *physical* set-piece.
- **Composed, not scripted** — the jackknife is the trailer physics + the AI action
  ([C46.5](../C46-AI-Goals-Actions/05-action-menu.md)) + the roadblock system
  ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) — composed from existing
  systems, the engine's recurring economy ([C61.4](../C61-Traffic-Ambient/04-traffic-behavior.md)).

So the jackknife is where hard physics ([C62.3](03-trailers.md)) becomes memorable gameplay — the articulated
trailer, weaponised into a rolling roadblock, composed from the constraint system, the AI, and the pursuit. It's a
small feature, but it embodies the book's recurring lessons: real physics
([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)), composed from reusable systems, producing
emergent spectacle. Reading the jackknife ties the linked-body physics to the pursuit's drama.

## RE implications

- **The `JackKnife`** (`AIActionJackKnife`) turns a truck-and-trailer into a pursuit roadblock — the articulation
  weaponised.
- **A dynamic roadblock** — a truck actively swinging its trailer across the road, vs. pre-parked cruisers.
- **Physics as spectacle** — the hardest-to-simulate body (trailer) enables the most spectacular obstacle
  (jackknife).
- **Composed** — trailer physics + AI action + roadblock system — not scripted, the engine's economy.

---

### Key takeaways

- **`JackKnife`** (via **`AIActionJackKnife`**) turns a truck-and-trailer into a **pursuit roadblock** — a
  cop-directed semi swinging its trailer across the road to block you — the articulated physics **weaponised**.
- It's a **dynamic** roadblock — a truck *actively* swinging its trailer in motion — vs. the static roadblock's
  pre-parked cruisers ([C49.4](../C49-Cops-Dispatch-Roadblocks/04-roadblocks.md)) — more dramatic and
  unpredictable.
- The engine's investment in **trailer stabilisation** (`SuspensionTrailer`, 99 methods,
  [C62.3](03-trailers.md)) pays off — a *controlled* jackknife needs exactly that code.
- The jackknife exemplifies **physics as spectacle** — the **hardest-to-simulate body enables the coolest
  obstacle**; the difficulty *is* the payoff.
- It's **composed** (trailer physics + AI action + roadblock system), not scripted — real, interactive physics, the
  engine's recurring economy.

**Continue:** [C62.5 — Reading constraints in RE](05-reading-constraints.md) · [Chapter 62 hub](C62-Constraints-Joints.md)
