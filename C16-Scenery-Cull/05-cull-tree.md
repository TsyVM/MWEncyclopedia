# C16.5 — The Cull Tree

> **The one-sentence version:** `SceneryTreeNodes` is a per-section AABB tree with fanout ≤ 5 whose 36-byte
> nodes hold a box (`+0x00`/`+0x0C`) and five `i16` entries (`+0x1A`): non-negative = an instance index,
> negative = a child node; node 0 is the root, and the leaves partition every instance exactly once.

[← C16.4 — The 72-byte SceneryInfo](04-info-record.md) · [Chapter 16 hub](C16-Scenery-Cull.md) ·
[Next: C16.6 — Editing scenery safely →](06-editing-scenery.md)

---

## The node

The cull tree is a flat array of 36-byte nodes; the chunk *is* the runtime tree (the loader reads it directly
and computes the node count as `size / 36`). Each node:

| Offset | Type | Field |
|---|---|---|
| `+0x00` | `f32[3]` | AABB min (world, Z-up) |
| `+0x0C` | `f32[3]` | AABB max (world, Z-up) |
| `+0x1A` | `i16[5]` | five entries: `≥ 0` → **instance index** (into this section's instances); `< 0` → **child node** index = `−value` |

Node 0 is the **root**. You descend from it: for each of a node's five entries, a negative value points at a
child node to recurse into, and a non-negative value names an instance that lives in this node (a leaf entry).
The fanout is at most five — a shallow, wide tree rather than a deep binary one.

## The partition invariant

The property that makes the tree *correct* is that its leaf entries form an **exact partition** of the
section's instances: every instance is referenced by exactly one leaf — none missing, none duplicated.
Verified across **all 947 retail sections** (19,578 nodes, 77,783 leaf references): every non-root node is
referenced exactly once, child boxes nest inside their parents, and the leaves cover every instance once.

This invariant is what guarantees correct drawing: descend the tree, and each visible instance is emitted
exactly once. Break the partition and you get the two classic bugs:

- **An instance referenced twice** → drawn/tested twice (wasteful, possible z-fighting).
- **An instance referenced zero times** → never drawn — an invisible prop.

## How the tree is used

Culling with the tree is a recursive descent that replaces "test every prop":

```python
def cull(nodes, instances, node_index, frustum, out):
    node = nodes[node_index]
    if not frustum.intersects(node.aabb):
        return                                  # whole subtree culled — the big win
    for e in node.entries:                      # five i16 slots
        if e >= 0:
            if frustum.intersects(instances[e].aabb):
                out.append(e)                   # leaf: a potentially-visible instance
        elif e != EMPTY:
            cull(nodes, instances, -e, frustum, out)   # child node
```

If a node's box is outside the frustum, its **entire subtree** is skipped — one box test discards hundreds of
props. That is the whole point: a precomputed spatial hierarchy turns "test 77,783 boxes" into "descend a
handful of nodes." It is the per-section fine filter that complements the section-level potentially-visible
set from streaming ([C15.5](../C15-Track-Streaming/05-visibility.md)).

> ✅ *Verified:* 36-byte node stride; AABB at `+0x00`/`+0x0C`; five `i16` entries at `+0x1A` with the
> instance-vs-child sign convention; node 0 root; leaves partition the instances exactly, across all 947
> sections (77,783 leaf refs = 77,783 instances).

## Building or rebuilding the tree

Because nodes are fixed 36-byte records referenced by index, a rebuilt tree can be a different size than the
original — you regenerate it from the instances:

1. Compute each node's AABB as the union of its entries' boxes.
2. Split top-down until every node holds ≤ 5 entries.
3. Emit node 0 as the root; children as negative indices.
4. Keep **every instance in exactly one leaf** — the retail invariant.

There is no loader-imposed depth limit (retail trees go up to depth 13). The one hard rule is the partition:
every instance in exactly one leaf, child boxes nested in parents.

## Editing implications

- **New or moved instances must be in the tree.** Add an instance ([C16.3](03-instance-record.md)) and it must
  appear in a leaf, or it never draws (pop-in / invisible). Move an instance and its containing node's box must
  still enclose it, or it may be wrongly culled.
- **Preserve the partition.** After any change, every instance is referenced by exactly one leaf.
- **Rebuild is fine.** Regenerating the whole tree from the instances is often simpler than patching it —
  the size can change, the invariant cannot.

---

### Key takeaways

- The cull tree is a flat array of 36-byte nodes: AABB (`+0x00`/`+0x0C`) + five `i16` entries (`+0x1A`).
- Entry sign: `≥ 0` = instance index (leaf), `< 0` = child node (`−value`); node 0 is the root; fanout ≤ 5.
- Leaves form an **exact partition** of instances — verified across 947 sections (one leaf ref per instance).
- Culling descends the tree, skipping whole subtrees whose box misses the frustum — the per-section fine
  filter.
- Edits must keep every instance in exactly one leaf and every node's box enclosing its contents; rebuilding is
  allowed.

**Continue:** [C16.6 — Editing scenery safely](06-editing-scenery.md) · [Chapter 16 hub](C16-Scenery-Cull.md)
