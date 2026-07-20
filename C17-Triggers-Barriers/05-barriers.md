# C17.5 — Barriers

> **The one-sentence version:** barriers are the invisible walls that keep the car inside the drivable world —
> collision geometry that *stops* you, as opposed to triggers that *fire an event* — stored as named 2-D
> "trough-boundary" polylines (chunk `0x00034190`) that wall the map edges, closed streets, and race limits.

[← C17.4 — The even–odd containment test](04-even-odd.md) · [Chapter 17 hub](C17-Triggers-Barriers.md) ·
[Next: C17.6 — Events, messages & editing →](06-events-editing.md)

---

## Barriers vs triggers

Both are invisible world features, but they do opposite things:

| | Trigger | Barrier |
|---|---|---|
| Purpose | fire a gameplay **event** on entry | **block** the car from passing |
| Mechanism | even–odd containment → message ([C17.4](04-even-odd.md)) | collision geometry |
| On contact | scripts react (checkpoint, speed trap) | the car is physically stopped/deflected |
| Player feels | nothing physical (an event happens) | a wall |

A trigger is permeable — you drive through it and something happens. A barrier is solid — you can't drive
through it. They are layered over the same world but serve the two halves of "gameplay boundaries": *notify*
and *confine*.

## What barriers bound

Barriers exist to keep the player where the designers want them:

- **Map edges.** The drivable world is finite; barriers wall its perimeter so you can't drive off into
  unstreamed void.
- **Closed streets.** Roads that exist visually but aren't part of the playable area are barriered off.
- **Race boundaries.** Some events constrain the route; barriers (or barrier-like limits) enforce the course.
- **Gameplay funnels.** Barriers channel pursuits and races along intended paths.

Without them, the streaming world ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) would let the
car reach areas with no geometry loaded — barriers are part of what makes the open world feel bounded and
intentional rather than a floating patch of city.

## Barriers as collision

Mechanically, a barrier is **collision geometry** — a surface the physics system treats as solid, deflecting
the car on contact ([surfaces and collision are the physical side of the world](../C14-Vault-Pursuit-Surfaces/03-surfaces.md)).
Unlike a trigger's even–odd footprint test, a barrier participates in the car's physics: the collision solver
resolves the car against it, producing the wall-scrape and stop you feel. So a barrier's "shape" is a
collision primitive, not a gameplay polygon.

## The barrier format: trough boundaries

The on-disk barrier is a **trough boundary** — a named 2-D polyline that walls the drivable "trough." Like triggers
([C17.1](01-footprints.md)), it's a *top-down* outline: in the Z-up world ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md))
the wall extends vertically, so only its ground-plane `(X, Y)` path needs storing. Each barrier segment is a chunk
(`0x00034190`) whose payload is a 116-byte preamble then the point array:

```
8-byte chunk header:  u32 magic = 0x00034190,  u32 payloadSize
116-byte preamble:
  +0x00  u8[8]   0            (two zero dwords)
  +0x08  char[80] name        (segment name; JOAAT-hashed for lookup)
  +0x58  u8[8]   flags
  +0x60  f32[4]  bbox2D        (minX, minY, maxX, maxY)
  +0x70  u32     pointCount
then pointCount x { f32 x, f32 y }   // the 2-D barrier polyline
payloadSize == 116 + pointCount * 8   (self-describing)
```

So a barrier is a *named, bounded polyline* — a wall traced as a chain of ground-plane points, with a 2-D bounding
box for coarse rejection ([C17.2](02-trigger-record.md)) exactly like a trigger's gate. The name lets designers
address individual walls (a closed street, a race edge); the bbox lets the runtime skip segments far from the car
before testing the polyline. The physics then treats the polyline as the solid surface the car is deflected along
([Barriers as collision](#barriers-as-collision)).

> 🟡 *Verified against retail track data; not an `speed.exe` code literal.* The `0x00034190` segment layout
> (116-byte preamble — `name[80]`, `flags[8]`, 2-D bbox, `pointCount` — then `pointCount` × `vec2`, size
> self-checking as `116 + n·8`) parses cleanly across the retail world's barrier segments. Unlike the trigger
> chunks (`0x0003414A`/`0x00034146`, whose IDs appear as `cmp` dispatch immediates in `speed.exe`), `0x00034190`
> does **not** appear as a literal in the executable — the game consumes trough boundaries through a generic
> (non-literal-compare) path — so this format is tiered against the *data*, not the exe dispatch. The trigger side
> remains byte- and behaviour-verified ([C17.2](02-trigger-record.md)–[C17.4](04-even-odd.md)).

## Editing barriers

Because barriers are collision that confines the player, editing them changes where the car can go:

- **Removing a barrier** opens an area — useful for exploration mods, but risky: the area beyond may have no
  streamed geometry or road network ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)), so the
  car can reach a void or an un-navigable region.
- **Adding a barrier** confines the player — closing a shortcut, fencing a hazard.
- **Moving a barrier** reshapes the drivable boundary.

The caution is that barriers interlock with the rest of the world: an opened barrier can expose missing
scenery, absent triggers, or a road network that doesn't cover the newly-reachable area. Edit them knowing
what lies beyond.

## Barriers and the whole boundary system

Triggers and barriers together with the road network form the world's "where and what" for the car:

- **Barriers** say *where you can't go* (confine).
- **Triggers** say *what happens where you are* (notify).
- **The road network** ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) says *where the AI
  and GPS route*.

A coherent world edit considers all three: open a barrier and you may need to extend the road network and add
triggers to make the new area playable, not just reachable.

---

### Key takeaways

- Barriers **block** the car (collision); triggers **fire events** (even–odd footprint) — opposite roles over
  the same world.
- On disk a barrier is a **named 2-D trough-boundary polyline** (chunk `0x00034190`): a 116-byte preamble
  (`name[80]`, flags, 2-D bbox, `pointCount`) then `pointCount` × `(x, y)` ground-plane points — verified against
  retail track data (the ID isn't an `speed.exe` code literal, unlike the trigger chunks).
- Barriers bound map edges, closed streets, race limits, and funnel gameplay — they make the open world feel
  intentional.
- A barrier is collision geometry the physics solver resolves against, not a gameplay polygon.
- Editing barriers changes where the car can go; opening one can expose missing geometry/road network beyond.
- Barriers (confine), triggers (notify), and the road network (route) together define the world for the car —
  edit them coherently.

**Continue:** [C17.6 — Events, messages & editing](06-events-editing.md) · [Chapter 17 hub](C17-Triggers-Barriers.md)
