# C64.4 — World Animations

> **The one-sentence version:** the `WorldAnim*` system drives the world's ambient animations — flapping flags,
> swaying signs, working machinery — with `WorldAnimEntityTree` organising them spatially, `WorldAnimTrigger`
> firing them by proximity, and `WorldAnimCtrl` driving them, ticked in the world update.

[← C64.3 — One-shot effects](03-one-shot-effects.md) · [Chapter 64 hub](C64-World-Update.md) ·
[Next: C64.5 — Reading the world update in RE →](05-reading-world-update.md)

---

## Ambient animation: the living scenery

Beyond the cars ([C64.2](02-active-body-list.md)) and effects ([C64.3](03-one-shot-effects.md)), the world has
*ambient animation* — the non-interactive moving scenery that makes the city feel alive
([Chapter 26](../C26-World-Ambient-Animation/C26-World-Ambient-Animation.md)). The verified **`WorldAnim*`** system
(7 classes) drives it:

| Class | Role |
|---|---|
| `WorldAnimations` | the ambient-animation system |
| `WorldAnimEntity` | an animated world entity |
| `WorldAnimEntityTree` | the spatial organisation of animated entities |
| `WorldAnimInstanceEntry` | a placed animation instance |
| `WorldAnimTrigger` | fires animations by proximity/event |
| `WorldAnimCtrl` | drives the animations |

So the city's ambient motion — a flag flapping, a sign swaying, a machine working, a fountain flowing
([C59.2](../C59-Audio-Runtime/02-3d-positional.md), `SFXCTL_3DFountainPos`) — is `WorldAnim*` playing animations on
world entities. This is the *life* layer of the scenery ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)):
the static geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) plus its ambient animations.

> ✅ *Verified:* the `WorldAnim*` classes — `WorldAnimations`, `WorldAnimEntity`, `WorldAnimEntityTree`,
> `WorldAnimInstanceEntry`, `WorldAnimTrigger`, `WorldAnimCtrl` — are present in `speed.exe`.

## The EntityTree: spatial organisation

**`WorldAnimEntityTree`** (verified) is the *spatial organisation* of animated entities — a tree structure
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) that lets the world find and manage the animations *near*
the player efficiently:

- **Spatial culling** — only animations *near/visible* ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) need
  ticking; the tree finds them, so distant animations don't cost update time.
- **Proximity triggering** — `WorldAnimTrigger` ([below](#triggers-proximity-activation)) uses the tree to fire
  animations as the player approaches.

So `WorldAnimEntityTree` is to *animations* what the scenery cull tree ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md))
is to *rendering* — a spatial structure that scopes the work to the player's vicinity. This is the *pay-for-what's-near*
economy again ([C61.2](../C61-Traffic-Ambient/02-traffic-density.md)): a whole city of potential animations, but
only the nearby ones ticked. The tree is what makes ambient animation *affordable* at city scale — you don't
animate the flags across town, only the ones around you.

> 🟡 *Reasoned:* the spatial-culling role of `WorldAnimEntityTree` is the natural reading of a spatial "EntityTree"
> for animations, consistent with the scenery cull tree ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) and
> the pay-for-what's-near economy; the exact tree structure is deeper RE. The `WorldAnim*` classes are verified.

## Triggers: proximity activation

**`WorldAnimTrigger`** (verified) *fires* animations by proximity or event — the mechanism that starts an animation
when relevant ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)):

- **Proximity** — an animation starts when the player *approaches* — a machine begins working as you near it, so
  it's animating *when you can see it* (and idle/culled when you can't, [above](#the-entitytree-spatial-organisation)).
- **Event** — some animations fire on *events* ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) — a scripted
  moment, a pursuit beat.

So triggers ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)) are how ambient animations *activate*
— not all running always, but firing when the player is near or an event occurs. This is efficient (only relevant
animations run) and *responsive* (the world seems to react to your presence — the machine that starts as you arrive
feels alive). `WorldAnimCtrl` then *drives* the triggered animation (advancing its keyframes,
[Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)). Together, `WorldAnimTrigger` (when) + `WorldAnimCtrl`
(drive) + `WorldAnimEntityTree` (where) are the ambient-animation system — the living-scenery layer of the world,
ticked in `WorldUpdate` ([C64.1](01-world-tick.md)) alongside the bodies and effects.

## Why ambient animation matters

The `WorldAnim*` system ([above](#ambient-animation-the-living-scenery)) is a subtle but important contributor to
the world's feel:

- **Life** — a *static* city (buildings that never move) feels dead; ambient animation (flags, signs, machinery)
  makes it feel *inhabited and working*, like traffic ([Chapter 61](../C61-Traffic-Ambient/C61-Traffic-Ambient.md))
  does for the roads.
- **Detail** — small motions (a swaying sign, a flapping banner) add richness that rewards attention — the world
  *rewards looking*.
- **Responsiveness** — proximity triggers ([above](#triggers-proximity-activation)) make the world seem to *react*
  to you (things start as you arrive), a subtle liveliness.

So ambient animation is the *scenery's* version of the life that traffic gives the roads
([Chapter 61](../C61-Traffic-Ambient/C61-Traffic-Ambient.md)) — the moving details that make Rockport a *place*
rather than a set of models. It's ticked in the world update ([C64.1](01-world-tick.md)) as a first-class content
system, scoped to the player's vicinity ([above](#the-entitytree-spatial-organisation)) for affordability. Reading
`WorldAnim*` completes the world-update picture: the world advances its *bodies* (cars), its *effects* (explosions),
*and* its *animations* (living scenery) — the three kinds of moving content, all ticked each frame.

## RE implications

- **`WorldAnim*`** drives the world's ambient animations — flags, signs, machinery (the living scenery).
- **`WorldAnimEntityTree`** organises them spatially — only nearby animations tick (pay-for-what's-near).
- **`WorldAnimTrigger`** fires them by proximity/event; **`WorldAnimCtrl`** drives them.
- **Ambient animation** gives the scenery *life* — the counterpart of traffic for the roads — ticked in the world
  update.

---

### Key takeaways

- The **`WorldAnim*`** system (7 classes) drives the world's **ambient animations** — flapping flags, swaying
  signs, working machinery — the **living-scenery** layer.
- **`WorldAnimEntityTree`** organises animated entities **spatially** — only nearby/visible animations tick (the
  pay-for-what's-near economy, like the scenery cull tree).
- **`WorldAnimTrigger`** fires animations by **proximity/event** (a machine starts as you approach); **`WorldAnimCtrl`**
  drives the triggered animation.
- Ambient animation gives the **scenery life** — the counterpart of traffic for the roads — making Rockport a
  *place*, not a set of models.
- It's ticked in **`WorldUpdate`** alongside bodies and effects — the world advances its **bodies, effects, and
  animations**, the three kinds of moving content.

**Continue:** [C64.5 — Reading the world update in RE](05-reading-world-update.md) · [Chapter 64 hub](C64-World-Update.md)
