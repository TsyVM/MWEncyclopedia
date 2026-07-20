# C18.4 — The Graph: Adjacency & Roads

> **The one-sentence version:** nodes and segments form a navigable graph that *closes exactly* — every
> segment names two valid nodes, and Σ node-degree = 2 × segments — while `RNrd` roads group segments into the
> named streets the GPS and map use.

[← C18.3 — The RNsg segment](03-segment-record.md) · [Chapter 18 hub](C18-Road-Network-CARP.md) ·
[Next: C18.5 — Routing: cops, traffic & GPS →](05-routing.md)

---

## Vertices and edges

The road network is a graph in the textbook sense:

- **Vertices** = `RNnd` nodes ([C18.2](02-node-record.md)) — points in the world.
- **Edges** = `RNsg` segments ([C18.3](03-segment-record.md)) — roads joining two nodes.

Navigating the world is walking this graph: from a node, its segment list gives the roads out; each segment
leads to another node; repeat. Everything that routes — traffic, cops, GPS — does some version of this walk
([C18.5](05-routing.md)).

## Closure is the correctness proof

The decode is validated by a global invariant: **the graph closes exactly.** Concretely,

```
Σ over all nodes of node.degree  ==  2 × segmentCount
```

Every segment has two endpoints, so it appears in exactly two nodes' segment lists; summing degrees counts
each segment twice. Verified on the retail graph: the equality holds exactly, every segment's endpoints are
valid node ordinals, and the node ordinals run 0…N−1 with no gaps. This closure is powerful evidence that the
32-byte node and 22-byte segment layouts are correct — a wrong stride or offset would break the count
immediately.

```python
def verify_closure(nodes, segments):
    deg_sum = sum(n["degree"] for n in nodes)
    assert deg_sum == 2 * len(segments), "graph does not close — decode error"
    for s in segments:
        assert s["a"] < len(nodes) and s["b"] < len(nodes), "dangling segment endpoint"
    for k, n in enumerate(nodes):
        assert n["index"] == k, "node ordinal != position"
```

Running these three checks on any road blob tells you instantly whether your parse is sound — the road-network
equivalent of "offsets tile the blob" ([C5.5](../C5-Textures-TPK/05-extract-replace.md)) or the mesh's three
identities ([C9.6](../C9-Meshes-FVF/06-assembling.md)).

## Roads group segments

`RNrd` is the **road** layer — 1,308 roads in the worked track — grouping segments into named streets. A road
is the human-facing unit: "Rosewood Drive" is a road made of many segments and nodes. Nodes carry a `roadIndex`
([C18.2](02-node-record.md)) linking them to their road, so:

- **The fine graph** (nodes + segments) is what routing walks.
- **The road grouping** (`RNrd`) is what the GPS labels and the map draws.

This two-level structure mirrors the world's other groupings (scenery instances grouped by info, streaming
sections grouped by grid): a fine mechanism layer and a coarse organisational layer over it.

## Undirected topology, directed travel

The graph's *topology* is undirected — a segment connects A and B, and both nodes list it. But *travel* over it
is directed: one-way streets, divided roads, and preferred senses mean the cost A→B can differ from B→A. That
directionality lives in the segment flags ([C18.3](03-segment-record.md)), which are the frontier
([C18.6](06-frontier-editing.md)). So you can fully reconstruct the graph's *shape* today; its *traffic rules*
are partially open.

## What you can do with the graph

Even with flags partially decoded, the closed graph supports real analysis:

- **Reconstruct the road map.** Node positions + segments draw the entire drivable road network — the data
  behind the minimap ([Chapter 29 in the plan](../README.md)) and the GPS.
- **Measure connectivity.** Degrees reveal intersections (high degree) vs. mid-road points (degree 2).
- **Compute routes** with arc-length costs ([C18.5](05-routing.md)) — approximate until one-way flags are
  fully modelled.

> ✅ *Verified:* the graph closes exactly (Σ degree = 2 × segments), endpoints valid, ordinals contiguous —
> across the retail graph (4,385 nodes, 6,538 segments).
> 🟡 *Reasoned/open:* directed-travel rules (one-way, class) depend on segment flags still being decoded.

---

### Key takeaways

- Nodes (vertices) + segments (edges) = a navigable graph; navigation is walking node → segment → node.
- The graph **closes exactly**: Σ node-degree = 2 × segments, all endpoints valid, ordinals contiguous — the
  decode's correctness proof.
- `RNrd` roads group segments into named streets — the GPS/map layer over the fine graph.
- Topology is undirected; travel is directed (one-way/class) via segment flags, which are the frontier.
- You can reconstruct the road map and measure connectivity now; exact directed routing awaits the flags.

**Continue:** [C18.5 — Routing: cops, traffic & GPS](05-routing.md) · [Chapter 18 hub](C18-Road-Network-CARP.md)
