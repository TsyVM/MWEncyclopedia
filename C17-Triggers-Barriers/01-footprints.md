# C17.1 — Triggers as 2-D Footprints

> **The one-sentence version:** because the world is Z-up and the car drives on a surface, a gameplay volume
> only needs a top-down polygon on the X–Y plane — so triggers are cheap 2-D footprints with an implied
> vertical extent, not full 3-D volumes.

[← Chapter 17 hub](C17-Triggers-Barriers.md) · [Next: C17.2 — The trigger record →](02-trigger-record.md)

---

## Why 2-D is enough

The world is **Z-up** ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)): Z is height, and cars drive on the
ground. So "is the car inside this gameplay region?" almost never depends on altitude — it depends on where the
car is on the *map*. A trigger therefore doesn't need a 3-D shape; it needs a **footprint** on the horizontal
plane, and the question becomes a 2-D point-in-polygon test ([C17.4](04-even-odd.md)).

This is the same insight that makes the streaming grid 2-D ([C15.4](../C15-Track-Streaming/04-world-grid.md)):
in a driving game, the interesting geometry of *where things are* lives in the ground plane, and height is a
secondary concern. Triggers lean on it fully.

## What the footprint saves

Representing a gameplay volume as a 2-D polygon rather than a 3-D mesh buys a lot:

- **Cheap containment.** A point-in-polygon test over a handful of edges beats a 3-D volume test, and the
  engine runs it constantly (every trigger, every frame the car might be inside).
- **Compact storage.** A polygon is a list of 2-D points; a 3-D volume is faces and normals. A checkpoint is a
  few vertices.
- **Easy authoring.** Designers draw regions top-down on the map — the natural way to lay out a race route,
  place checkpoints, or fence off an area.

The trade — no vertical discrimination — is almost always irrelevant, because the car is on the road. Where
height *does* matter (a multi-level interchange), the coarse gate and careful polygon placement handle it.

## The AABB is a ground-plane box

Consistent with the 2-D model, a trigger's bounding box is stored as **`(minX, minZ, maxX, maxZ)`** — note it
pairs X with Z, the two horizontal axes as the trigger system treats them, giving a rectangle in the drivable
plane ([C17.2](02-trigger-record.md)). The polygon's vertices live in that plane; the box is their tight
bound, used as a fast reject before the exact test.

> ✅ *Verified:* a real trigger's AABB decodes to a ground-plane rectangle (`(4296.4, 1436.6, 4374.8, 1584.7)`)
> with an 8-vertex polygon — a 2-D footprint, exactly as the model predicts.

## An implied vertical extent

"2-D" doesn't mean "infinitely thin" — a trigger is a footprint *extruded* vertically enough to catch a car
driving through it. Conceptually it's a prism: the polygon is the cross-section, and any car within the
polygon's X–Y footprint (at road height) is inside. The engine doesn't test a Z range for most triggers; being
within the footprint is being inside. This is why a trigger works whether the car is low-slung or tall — the
footprint is what counts.

## Consequences for editing

Thinking of triggers as top-down polygons shapes how you edit them ([C17.6](06-events-editing.md)):

- **Place and reshape in the plane.** Move a checkpoint by shifting its polygon's X–Y vertices; grow a region
  by adding points to its footprint.
- **Keep the AABB tight.** The box must bound the polygon; after moving vertices, recompute
  `(minX, minZ, maxX, maxZ)` ([C17.2](02-trigger-record.md)).
- **Don't worry about height** for ordinary ground triggers — the footprint is the region.

---

### Key takeaways

- A Z-up world makes gameplay volumes 2-D: a trigger is a top-down polygon footprint, not a 3-D mesh.
- The footprint saves containment cost, storage, and authoring effort, at the near-costless price of no
  vertical discrimination.
- The AABB is a ground-plane rectangle `(minX, minZ, maxX, maxZ)`; the polygon vertices live in that plane.
- A trigger is effectively an extruded prism — being in the footprint (at road height) is being inside.
- Edit triggers by reshaping their planar polygons and keeping the AABB tight; height is rarely a concern.

**Continue:** [C17.2 — The trigger record](02-trigger-record.md) · [Chapter 17 hub](C17-Triggers-Barriers.md)
