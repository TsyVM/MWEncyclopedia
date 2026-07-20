# C18.3 — The RNsg Segment

> **The one-sentence version:** each segment is a 22-byte edge joining two endpoint nodes (`nodeA`, `nodeB`),
> carrying the road's arc length (stored scaled ×64) and flags — the graph's edges to the nodes' vertices.

[← C18.2 — The RNnd node](02-node-record.md) · [Chapter 18 hub](C18-Road-Network-CARP.md) ·
[Next: C18.4 — The graph: adjacency & roads →](04-graph.md)

---

## The record

`RNsg` is a flat array of `count` segments (6,538 in the worked track). Each segment is **22 bytes**:

| Offset | Type | Field |
|---|---|---|
| `+0x00` | `u16` | **nodeA** (endpoint node ordinal) |
| `+0x02` | `u16` | **nodeB** (endpoint node ordinal) |
| `+0x04` | (scaled) | **arc length** (road length between the nodes, stored ×64) |
| `+0x06`… | flags / geometry | segment flags and shape data |

A segment is an **edge**: it connects two nodes and describes the road between them. The two endpoints are node
ordinals ([C18.2](02-node-record.md)), so a segment is `node[nodeA] ── road ── node[nodeB]`. Both endpoints
must be valid node indices — verified across all 6,538 retail segments.

## Arc length is stored scaled

The arc length — the actual road distance between the two nodes, which may be longer than the straight-line
distance if the road curves — is stored as a **fixed-point value scaled by 64** (so the real length is the
stored value ÷ 64). Scaled integer storage is a common EA space-saving choice: a `u16`-range fixed-point number
covers road lengths to fine precision without a full float. Routing uses arc length as the base cost of
traversing a segment ([C18.5](05-routing.md)) — a longer road costs more to drive.

## The endpoints tie back to the nodes

The segment's `nodeA`/`nodeB` and the nodes' segment lists are two views of the same adjacency, and they must
agree:

- Segment *s* names nodes A and B.
- Node A's `segList` includes *s*; node B's includes *s*.

This mutual reference is what makes the graph *closed* and traversable in both directions: from a node you find
its segments (node → segList), and from a segment you find its nodes (segment → nodeA/nodeB). The verified
closure — `Σ node degree = 2 × segmentCount` ([C18.4](04-graph.md)) — is exactly this agreement counted
globally.

```python
def read_segments(blob, count, off):
    segs = []
    for s in range(count):
        r = blob[off + s*22 : off + s*22 + 22]
        node_a, node_b = struct.unpack_from("<2H", r, 0)
        arc_len_raw = u16(r, 0x04)          # ÷64 for real length
        segs.append({"a": node_a, "b": node_b, "arc_len": arc_len_raw / 64.0, "raw": r})
    return segs
```

> ✅ *Verified:* the 22-byte stride and the segment count (6,538) are confirmed on the retail blob; every
> segment's endpoints are valid node ordinals, and the node/segment adjacency closes exactly.
> 🟡 *Reasoned/open:* the precise semantics of the segment **flags** (lane count, direction, one-way, road
> class) are the frontier — identified as flags but not fully decoded ([C18.6](06-frontier-editing.md)).

## Direction and one-way roads

A segment connects two nodes, but roads have direction — one-way streets, divided highways, preferred travel
sense. That information lives in the segment's flags (still partially decoded). This matters for routing: a
one-way segment is traversable in only one direction, so the cost from A→B differs from B→A. Until the flags
are fully decoded, treat direction/one-way behaviour as data you read cautiously rather than rewrite
([C18.6](06-frontier-editing.md)).

## Editing implications

- **Both endpoints must be valid nodes.** A segment naming a non-existent node ordinal dangles — routing breaks
  or crashes.
- **Keep the adjacency mutual.** If you add a segment, add it to both endpoint nodes' segment lists and bump
  their degrees ([C18.2](02-node-record.md)); if you remove one, remove it from both.
- **Arc length is scaled ×64.** Write the real length × 64; a wrong scale makes a road seem 64× too short or
  long to the router.
- **Flags are risky.** Don't rewrite segment flags you haven't decoded — direction and road-class bits govern
  routing globally.

---

### Key takeaways

- An `RNsg` segment is 22 bytes: two endpoint node ordinals (`nodeA`, `nodeB`), an arc length (stored ×64), and
  flags.
- Segments are the graph's **edges**; both endpoints must be valid nodes (verified across 6,538 segments).
- Arc length is fixed-point ÷64 and is the base routing cost of a segment.
- Segment endpoints and node segment-lists are mutual — their agreement is the graph closure.
- Segment **flags** (direction/one-way/class) are the decode frontier — read cautiously, don't rewrite blindly.

**Continue:** [C18.4 — The graph: adjacency & roads](04-graph.md) · [Chapter 18 hub](C18-Road-Network-CARP.md)
