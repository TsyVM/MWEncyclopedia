# C18.2 — The RNnd Node

> **The one-sentence version:** each road node is a 32-byte graph vertex — a world position `(x, h, y)`, its
> ordinal (`nodeIndex` = array position), a road index into `RNrd`, a degree, and a list of up to seven
> neighbour-segment indices padded with `0xAAAA`.

[← C18.1 — CARP is a TLV blob](01-carp-format.md) · [Chapter 18 hub](C18-Road-Network-CARP.md) ·
[Next: C18.3 — The RNsg segment →](03-segment-record.md)

---

## The record

`RNnd` is a flat array of `count` nodes (4,385 in the worked track). Each node is **32 bytes**:

| Offset | Type | Field |
|---|---|---|
| `+0x00` | `f32` | position **x** |
| `+0x04` | `f32` | position **h** (height, Z-up) |
| `+0x08` | `f32` | position **y** |
| `+0x0C` | `u16` | **nodeIndex** (ordinal — equals the node's array position) |
| `+0x0E` | `u16` | **roadIndex** → `RNrd` |
| `+0x10` | `u8` | **degree** (number of segments meeting here) |
| `+0x11`… | `u16[7]` | **segList** — neighbour segment indices, `0xAAAA`-padded to fill 32 bytes |

The node is a graph vertex with a location and its local connectivity. The `0xAAAA` padding of the segment
list is a verified fingerprint — unused seg-list slots hold `0xAAAA`, which you saw directly in the retail
node records.

## The ordinal is the identity

`nodeIndex` at `+0x0C` equals the node's position in the array — node *k* has `nodeIndex = k`. Verified: the
ordinals run 0…4,384 with no gaps. This redundancy (storing the index that equals the position) is a
consistency anchor: a reader can assert `node[k].nodeIndex == k` and immediately catch a mis-strided parse.
Segments reference nodes by this ordinal ([C18.3](03-segment-record.md)), so it is the node's identity across
the graph.

## The segment list closes the graph

The `segList` names the segments that meet at this node — up to seven, with `degree` saying how many are real
and the rest `0xAAAA`. This is the node's side of the adjacency: node *k* lists its segments; each of those
segments names *k* as one endpoint. The two views agree, and their agreement is the graph-closure proof
([C18.4](04-graph.md)): summing every node's degree counts each segment twice (once per endpoint), giving
`Σ degree = 2 × segmentCount` — verified exact on the retail graph.

```python
def read_nodes(blob, count, off):
    nodes = []
    for k in range(count):
        r = blob[off + k*32 : off + k*32 + 32]
        x, h, y = struct.unpack_from("<3f", r, 0)
        node_index, road_index = struct.unpack_from("<2H", r, 0x0C)
        degree = r[0x10]
        seglist = [s for s in struct.unpack_from("<7H", r, 0x11) if s != 0xAAAA][:degree]
        nodes.append({"pos": (x, h, y), "index": node_index,
                      "road": road_index, "degree": degree, "segs": seglist})
    return nodes
```

> ✅ *Verified:* the 32-byte stride, the `0xAAAA`-padded segment list, and the node count (4,385) are confirmed
> on the retail blob; the field layout was decoded against the full graph with the ordinals running 0…4,384 and
> the adjacency closing exactly.

## Position is where the road point is

The `(x, h, y)` position places the node in the world (Z-up, so `h` is height —
[C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)). Nodes sit at intersections and along roads; the segments
between them ([C18.3](03-segment-record.md)) carry the road's shape. So the graph is geometric as well as
topological — routing can measure real distances, and the GPS line can be drawn through node positions.

## The road index groups nodes

`roadIndex` at `+0x0E` links the node to an `RNrd` road — the named street it belongs to. This is what lets the
GPS say "turn onto Rosewood Drive" and the map label roads: nodes and segments are the fine graph, roads are
the human-facing grouping over them ([C18.4](04-graph.md)).

## Editing implications

- **Preserve the ordinal invariant.** If you add/remove nodes, `nodeIndex` must still equal array position, or
  every segment reference breaks.
- **Keep degree and segList consistent.** `degree` must match the count of non-`0xAAAA` entries, and each
  listed segment must name this node — the closure invariant ([C18.4](04-graph.md)).
- **Positions are world Z-up.** Move a node and the segments to it change geometry; routing distances shift.
- **This is expert territory.** Because nodes, segments, and roads cross-reference by index, partial edits
  break routing globally ([C18.6](06-frontier-editing.md)).

---

### Key takeaways

- An `RNnd` node is 32 bytes: position `(x, h, y)`, `nodeIndex` (= array position), `roadIndex`, `degree`, and a
  `0xAAAA`-padded 7-slot segment list.
- The ordinal equals the array position (verified 0…4,384) — a consistency anchor and the node's identity.
- The segment list is the node's adjacency; it agrees with the segments, closing the graph (Σ degree = 2 ×
  segments).
- Position places the node in the Z-up world; `roadIndex` groups it into a named `RNrd` road.
- Edits must preserve the ordinal invariant and degree/segList consistency; graph editing is expert territory.

**Continue:** [C18.3 — The RNsg segment](03-segment-record.md) · [Chapter 18 hub](C18-Road-Network-CARP.md)
