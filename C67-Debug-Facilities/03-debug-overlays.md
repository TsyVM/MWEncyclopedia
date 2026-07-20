# C67.3 — Debug Overlays

> **The one-sentence version:** the debug overlays — `OL_CollisionDetection`/`OL_CollisionDetectionWidget` — *draw
> the invisible*, visualizing internal engine state (the collision world's AABBs and contacts) on screen so the
> developers could *see* the physics they were tuning.

[← C67.2 — Debug output & assertions](02-debug-output.md) · [Chapter 67 hub](C67-Debug-Facilities.md) ·
[Next: C67.4 — Debug cameras & screens →](04-debug-cameras-screens.md)

---

## Drawing the invisible

Text output ([C67.2](02-debug-output.md)) shows you *numbers*; a **debug overlay** shows you *geometry* — it draws
internal engine state *on screen*, spatially, so the developers could *see* it. `speed.exe` names the collision
overlay:

- **`OL_CollisionDetection`** — the collision-detection overlay: draws the collision world
  ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)) — the broad-phase AABBs
  ([C63.2](../C63-Collision-World/02-broad-phase.md)), the narrow-phase contacts
  ([C63.3](../C63-Collision-World/03-narrow-phase.md)), the collision geometry.
- **`OL_CollisionDetectionWidget`** — the *widget* form: a FEng widget
  ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) that hosts the overlay, drawn like any UI element.

The collision system ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) is *normally invisible* —
the AABBs, the contact points, the collision meshes are internal data the player never sees. The overlay *draws
them* — so a developer tuning collision could *see* the boxes the broad-phase tests, the points where contacts
form, the geometry the cars hit. This makes an invisible system *visible* — the single most valuable thing for
debugging spatial code. You can't debug collision by reading numbers; you have to *see* the boxes.

> ✅ *Verified:* `OL_CollisionDetection` and `OL_CollisionDetectionWidget` are present in `speed.exe` — the
> collision-detection debug overlay (among 61 `OL_*` overlay/screen entries).

## OL_ — the overlay/screen family

`OL_CollisionDetection` is one of a family — **`OL_*`** — 61 entries in `speed.exe`. The `OL_` prefix marks
*overlay* screens ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) — screens drawn *over* the game, including
both **online** menus (`OL_` = OnLine, the multiplayer/EA-online screens) and **debug overlays** like the collision
visualizer:

- **Online screens** — the LAN/online lobby, matchmaking, and profile screens — the `OL_` family is largely the
  online UI.
- **Debug overlays** — `OL_CollisionDetection` — dev visualizations drawn over the game.

So the `OL_` family mixes *online UI* and *debug overlays* — both are "screens drawn over gameplay," built on the
same FEng overlay system ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)). The collision overlay reuses the UI
overlay machinery to draw its visualization — the same `Widget` infrastructure ([C65.2](../C65-HUD-Runtime/02-gauge-cluster.md))
that draws the HUD draws the debug boxes. This is the engine's *uniformity* again ([C67.2](02-debug-output.md)):
debug visualization is *just another overlay*, built on the front-end system.

> 🟡 *Reasoned:* the split of the 61 `OL_*` entries into online-UI screens vs. debug overlays is the natural reading
> of the prefix and the verified `OL_CollisionDetection` overlay; the exact per-entry classification is RE. The
> `OL_*` count and the collision overlay are verified.

## Why visualize collision specifically

That the *collision* system got a dedicated overlay (`OL_CollisionDetection`) is telling — collision is the
*hardest* engine system to debug without visualization ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)):

- **It's spatial and invisible.** Collision is about *geometry in space* ([C63.3](../C63-Collision-World/03-narrow-phase.md))
  — boxes, points, meshes — that produces no visible output until something goes wrong (a car clips through a wall,
  gets stuck, bounces oddly). You can't see the *cause* without drawing the geometry.
- **It's where the physics feels wrong.** A racing game *lives or dies* on collision feel
  ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) — cars hitting walls, traffic, each other. Tuning
  that feel requires *seeing* what the collision system sees.
- **Bugs are subtle.** A missing contact ([C63.4](../C63-Collision-World/04-collision-cache.md)), a wrong AABB, a
  bad mesh — these are invisible in numbers but *obvious* when drawn (a box in the wrong place, a missing contact
  point).

So the collision overlay exists because collision is *the* system you can't debug blind — and a racing game's
quality *depends* on it ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)). The developers built
`OL_CollisionDetection` because they *needed to see* the collision world to make the driving feel right. Reading the
overlay confirms the collision system's *structure* ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)):
that it *has* AABBs and contacts to draw is the overlay telling you the system's shape — the debug tooling
documenting the system it instruments ([C67.1](01-debug-in-shipped.md)).

## Overlays as system documentation

Like assertions ([C67.2](02-debug-output.md)), debug overlays *document the system they draw*
([C67.1](01-debug-in-shipped.md)):

- **An overlay for X means X has drawable state.** `OL_CollisionDetection` existing means the collision system
  ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)) has *spatial data worth drawing* — AABBs, contacts —
  confirming its structure.
- **What the overlay draws names the data.** The overlay's draw code *references* the collision structures — so
  reading it (in RE) reveals *what* the collision world holds ([C63.1](../C63-Collision-World/01-collision-world.md)).
- **The overlay is a read path into the system.** To draw the collision world, the overlay must *read* it — so the
  overlay code is a *guided tour* of the collision data structures ([C67.5](05-reading-debug.md)).

So the debug overlay is *documentation you can execute* — it names, reads, and draws the system's internal state,
and reading its code (in RE) is a direct path into that state. The `OL_CollisionDetection` overlay is thus a *gift*
for understanding collision ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)): the developers wrote code
that *reads and displays* the collision world, and that code shipped ([C67.1](01-debug-in-shipped.md)) — a
ready-made map of the system, for anyone who reads it.

## RE implications

- **Debug overlays** (`OL_CollisionDetection`/`Widget`) draw *invisible* internal state — the collision world's
  AABBs and contacts — on screen.
- **The `OL_` family** (61 entries) mixes online UI and debug overlays — all FEng overlays
  ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)).
- **Collision got an overlay** because it's the hardest system to debug blind — spatial, invisible, quality-critical.
- **Overlays document their system** — an overlay for X means X has drawable state; the overlay code is a read path
  into it.

---

### Key takeaways

- **Debug overlays** — **`OL_CollisionDetection`/`OL_CollisionDetectionWidget`** — *draw the invisible*,
  visualizing the collision world ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)) (AABBs, contacts) on
  screen so developers could **see the physics** they tuned.
- The **`OL_` family** (61 entries in `speed.exe`) mixes **online UI** screens and **debug overlays** — both FEng
  overlays ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) drawn over gameplay, on the same `Widget`
  infrastructure as the HUD.
- **Collision got a dedicated overlay** because it's the **hardest system to debug blind** — spatial, invisible, and
  *quality-critical* for a racing game ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).
- **Overlays document their system** — an overlay for X means X has *drawable state*; the overlay's draw code is a
  **read path** into the system's internals, a ready-made map that shipped.
- Debug visualization is **just another overlay** — the engine's uniformity: the same front-end system draws the
  HUD, the online menus, and the debug boxes.

**Continue:** [C67.4 — Debug cameras & screens](04-debug-cameras-screens.md) · [Chapter 67 hub](C67-Debug-Facilities.md)
                                                                          