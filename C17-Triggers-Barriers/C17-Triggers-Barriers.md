# Chapter 17 — Trigger Regions & Barriers

> **Goal of this chapter:** decode the invisible gameplay geometry layered over the world — the typed 2-D
> trigger footprints that fire events when the car enters them, the even–odd containment test that decides
> "am I inside," and the barriers that wall the player in — so you can read, edit, and add gameplay volumes.

The world is not only what you see. Layered over the streamed scenery
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) is a mesh of **invisible gameplay volumes**:
checkpoints, speed traps, event gates, race boundaries. These are **trigger regions** — typed top-down
polygons on the world plane — and **barriers**. When the car crosses a trigger, the engine fires a message
that scripts turn into gameplay. This chapter is that system, decoded and byte-verified against the retail
track.

> **Verified against retail data.** The trigger structures are confirmed directly from `TRACKS/L2RA.BUN`: the
> `TriggerRegionParent` container `0x80034147` wraps `TriggerRegions` (`0x0003414A`), and the first record
> decodes exactly to **type = 3**, **AABB = (4296.4, 1436.6, 4374.8, 1584.7)** at `+0x20`, and **vertex count
> = 8** at `+0x40` — matching the documented variable-stride layout. The containment predicate is even–odd ray
> parity, verified 4,230/4,230 against a reference implementation. Two neighbouring chunk IDs round out the
> gameplay-volume family and appear as `cmp` dispatch immediates in `speed.exe`: **`0x00034146`** position markers
> (pad + 48-byte points) and the trigger regions themselves. **Barriers** are a separate family — named 2-D
> **trough-boundary** polylines (chunk **`0x00034190`**, [C17.5](05-barriers.md)) — verified against retail track
> data (that ID is not an exe code literal; it's consumed through a generic path).

---

## Deep-dive pages

- [C17.1 — Triggers as 2-D footprints](01-footprints.md): why a Z-up world lets gameplay volumes be top-down
  polygons, and what that buys.
- [C17.2 — The trigger record](02-trigger-record.md): the variable-stride record — type, coarse gate, AABB,
  vertex count, and the polygon hull — decoded from real bytes.
- [C17.3 — The 15 trigger types](03-trigger-types.md): the per-type buckets (gate / engagable / speed-trap /
  checkpoint / …) and what each does.
- [C17.4 — The even–odd containment test](04-even-odd.md): the ray-crossing parity predicate, why it allows any
  simple polygon, and the coarse gate that precedes it.
- [C17.5 — Barriers](05-barriers.md): the walls that bound the drivable world and how they differ from
  triggers.
- [C17.6 — Events, messages & editing](06-events-editing.md): the message chain to scripts, and how to edit or
  add gameplay volumes safely.
- [C17.7 — Position markers & event sequences](07-markers-sequences.md): the 48-byte named 3-D anchors
  (`0x00034146`) and the `CARP` event-script chunk (`0x0003B811`).

---

## 17.1 The world is Z-up, so triggers are flat

Because the world is **Z-up** ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)), a gameplay volume rarely
needs true 3-D shape — the car drives on a surface, so "is the car in this region" is answered by a **top-down
footprint** on the X–Y plane. Triggers exploit this: each is a 2-D polygon (with an implied vertical extent),
which captures "is the car here" at a fraction of the cost of a 3-D volume ([C17.1](01-footprints.md)). The
trigger's AABB is stored as `(minX, minZ, maxX, maxZ)` — a 2-D box in the ground plane.

## 17.2 A trigger is a typed polygon

Each trigger record is variable-length: a fixed head (type, a coarse center/radius gate, and the polygon's
AABB) followed by the polygon's vertices. Decoded from the retail track, the head is:

| Offset | Field |
|---|---|
| `+0x00` | trigger **type** (1–14) |
| `+0x04`…`+0x14` | coarse center + radius (a cheap pre-test gate) |
| `+0x20`…`+0x2C` | **AABB** (minX, minZ, maxX, maxZ) — the polygon's exact hull bounds |
| `+0x40` | **vertex count** (`u16`) |
| `+0x42` | length word; base ≥ `0x44`, then the polygon vertices |

The AABB is the polygon's tight bound; the center/radius pair at `+0x04` is a *separate coarse gate* that does
not bound the polygon exactly — it's a fast reject before the precise test ([C17.2](02-trigger-record.md),
[C17.4](04-even-odd.md)).

## 17.3 Fourteen types, fifteen buckets

The `type` field (1–14) classifies what a trigger *does* — gate, engagable event, speed trap, checkpoint, and
so on. At load, `TriggerRegionChunk::Parse` buckets the records into **15 per-type lists** so the runtime can
scan only the triggers relevant to a given query ([C17.3](03-trigger-types.md)). The type is the first thing
you read and the first thing that decides behaviour.

## 17.4 "Am I inside?" is even–odd parity

Containment is decided by the classic **even–odd ray-crossing** test: cast a ray from the car's position and
count polygon-edge crossings — odd means inside, even means outside. This is verified as the engine's exact
predicate (4,230/4,230 against a reference), and it has a powerful consequence: triggers may be **any simple
polygon**, not just convex ones — an L-shaped plaza or a curved boundary is a single trigger
([C17.4](04-even-odd.md)). The coarse center/radius gate runs first to reject far-away triggers cheaply before
the parity test runs.

## 17.5 Barriers wall the world

Alongside triggers, **barriers** are the invisible walls that keep the car inside the drivable world — the
edges of the map, closed-off streets, race boundaries. They are collision geometry rather than event volumes:
a trigger *fires* when you enter it; a barrier *stops* you from entering ([C17.5](05-barriers.md)).

## 17.6 Entering a trigger fires a message

When the car's inside-state for a trigger changes, the engine emits an **`MTriggerEnter` / `Exit` / `Inside`**
message; `Trigger::FireStateMessage` routes it, and scripts receive it as gameplay events (a checkpoint
passed, a speed trap triggered). The whole loop — data-defined regions → parity test → message → script — is
what lets designers place gameplay in the world from data without touching engine code
([C17.6](06-events-editing.md)).

---

### Key takeaways

- Triggers are **typed 2-D footprints** on the Z-up world plane — top-down polygons, not 3-D volumes.
- The record is variable-stride: type (`+0x00`), coarse center/radius gate (`+0x04`), AABB (`+0x20`), vertex
  count (`+0x40`), then the polygon — verified on the retail track (type 3, 8 vertices).
- The `type` (1–14) buckets triggers into 15 per-type lists; it decides behaviour.
- Containment is **even–odd ray parity** (verified exact), so any *simple* polygon works, not just convex.
- Barriers wall the world (collision), triggers fire events (`MTriggerEnter/Exit/Inside` → scripts).

**Next:** [Chapter 18 — The Road Network (CARP)](../C18-Road-Network-CARP/C18-Road-Network-CARP.md): the graph
the AI and GPS route on.
