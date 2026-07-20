# C16.6 — Editing Scenery Safely

> **The one-sentence version:** scenery is the friendly world edit — move, duplicate, or add props by editing
> 64-byte instances — provided you keep three invariants true: every info index in range, every instance's
> AABB matching its transform, and the cull tree a partition that includes every instance.

[← C16.5 — The cull tree](05-cull-tree.md) · [Chapter 16 hub](C16-Scenery-Cull.md) ·
[Next: Chapter 17 — Trigger Regions & Barriers →](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)

---

## The three invariants

Every scenery edit must preserve three things; violate any and the section mis-draws or crashes:

1. **Info index in range.** Each instance's `+0x3E` index must point at a real `SceneryInfo`
   ([C16.3](03-instance-record.md)).
2. **AABB matches the transform.** An instance's box (`+0x00`/`+0x0C`) must enclose the placed model at its
   transform ([C16.3](03-instance-record.md)).
3. **Cull tree is a partition covering all instances.** Every instance in exactly one leaf, every node's box
   enclosing its contents ([C16.5](05-cull-tree.md)).

Hold these three and scenery editing is safe and reversible.

## Move a prop

The most common edit, and the cleanest:

```python
def move_prop(instance, delta):
    instance.transform_pos = add(instance.transform_pos, delta)   # where
    instance.aabb_min = add(instance.aabb_min, delta)             # keep box with prop (inv. 2)
    instance.aabb_max = add(instance.aabb_max, delta)
    retree_if_needed(instance)                                    # keep tree valid (inv. 3)
```

Translation preserves extent, so the box just slides. If the prop moves out of its cull-tree node's box, its
node needs its box grown or the instance re-homed to a node that contains it — otherwise it's wrongly culled.
For small moves within a node this is automatic; for large moves, rebuild the affected subtree.

## Duplicate a prop

Copy an instance, give the copy a new transform, and register it in the tree:

```python
def duplicate_prop(instances, tree, src_index, new_transform, new_aabb):
    inst = copy(instances[src_index])            # same info index → same model
    inst.transform = new_transform; inst.aabb = new_aabb
    new_index = append(instances, inst)
    insert_into_leaf(tree, new_index, inst.aabb) # inv. 3: it must live in a leaf
    return new_index
```

The copy shares the source's info index, so it's the same model at a new place — a second lamppost. It must be
added to a cull-tree leaf, or it never draws.

## Add a new prop type

To place a model the section doesn't offer ([C16.2](02-models-instances.md)):

1. **Append a `SceneryInfo`** pointing at the solid (append — never insert — so indices don't shift,
   [C16.4](04-info-record.md)).
2. **Append `SceneryInstance`s** referencing the new info's index, each with a correct transform and AABB.
3. **Add them to the cull tree** (or rebuild it) so they're in leaves.

Order is load-bearing: info first (so the index exists), then instances, then tree.

## Remove a prop

Deleting an instance means removing it from the instance array *and* the cull tree, and — if indices shift on
removal — fixing every tree leaf and instance reference that moved. The simplest robust approach for bulk
removal is to mark-and-rebuild: null the doomed instances, then regenerate the instance array and the cull
tree together so all indices are consistent ([C16.5](05-cull-tree.md)).

## The size-tree still applies

Adding or removing records changes chunk sizes, so beyond the three scenery invariants the usual container
discipline applies ([C15.6](../C15-Track-Streaming/06-editing-track.md)): the section payload's EAGL size tree
must be fixed, and because sections are packed in the stream file, a size change re-stamps later section
offsets. Same-count edits (moving/re-modelling props without adding or removing) avoid this entirely — another
reason translation is the safe default.

## Verify

After a scenery edit, re-parse the section and confirm:

1. every instance's info index `< infoCount`;
2. every instance's AABB encloses its model at its transform (spot-check moved props);
3. the cull-tree leaves partition the instances (count leaf refs == instance count, no duplicates);
4. the section payload's size tree walks cleanly, and stream offsets are consistent if sizes changed.

Then drive the area: props appear where placed, fade at the right distance, and nothing pops or vanishes.

---

### Key takeaways

- Preserve three invariants: info index in range, AABB matching transform, cull tree a partition covering all
  instances.
- Move = shift transform + AABB by the same delta, then keep the tree valid; translation is the safe default.
- Duplicate = copy instance (shared info index) + new transform + add to a leaf; add-type = append info, then
  instances, then tree.
- Remove via mark-and-rebuild so indices stay consistent; size-changing edits pull in the section/stream
  size-tree fixups.
- Verify: indices in range, boxes enclosing, leaves partitioning, size tree clean — then drive it.

**Continue:** [Chapter 17 — Trigger Regions & Barriers](../C17-Triggers-Barriers/C17-Triggers-Barriers.md) ·
[Chapter 16 hub](C16-Scenery-Cull.md)
