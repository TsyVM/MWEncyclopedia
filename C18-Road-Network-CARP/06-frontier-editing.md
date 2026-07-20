# C18.6 — The Frontier: Cost Grids & Editing

> **The one-sentence version:** node and segment structure is solid, but the `CGrd`/`CGcn` cost grids, the
> segment flag semantics, and `RNpf`/`RNrd` internals are still partial — so you can *read and reason about*
> the graph confidently, but *rewriting* routing is expert territory.

[← C18.5 — Routing: cops, traffic & GPS](05-routing.md) · [Chapter 18 hub](C18-Road-Network-CARP.md) ·
[Next: Chapter 19 — Audio: Banks →](../C19-Audio-Banks/C19-Audio-Banks.md)

---

## What's decoded and what isn't

Honesty about the boundary of knowledge is part of a reference's job. For the road network:

| Piece | State |
|---|---|
| CARP container / tag directory | ✅ decoded ([C18.1](01-carp-format.md)) |
| `RNnd` nodes (32 B) | ✅ decoded ([C18.2](02-node-record.md)) |
| `RNsg` segments (22 B) — endpoints, arc length | ✅ decoded ([C18.3](03-segment-record.md)) |
| Graph closure / adjacency | ✅ verified ([C18.4](04-graph.md)) |
| Segment **flags** (one-way, class, lanes) | 🟡 partial |
| `CGrd` / `CGcn` **cost grids** | 🟡 partial |
| `RNpf` pathfind / `RNrd` road internals | 🟡 partial |

So the graph's *shape* is fully recoverable; its *traffic rules and costs* are the frontier.

## The cost grids

`CGrd` (cost grid) and `CGcn` (cost-grid connections) hold the **routing costs** the searches use
([C18.5](05-routing.md)) — beyond raw arc length, these encode how expensive it is to traverse parts of the
graph (road class, preferred routes, penalties). They are the least-decoded part of CARP, and they matter
because they, not just distance, shape the routes cops take and the line the GPS draws. Until they're fully
understood, you can observe their effect (routes that prefer certain roads) but not confidently rewrite them.

> 🟡 *Reasoned/open:* the `CGrd`/`CGcn` layout and the segment flag bit-meanings are identified as the cost and
> rule data but not fully decoded. This is the chapter's stated frontier.

## Why editing is expert territory

The road graph is a tightly cross-referenced structure, and its incompletely-decoded parts make edits risky:

- **Everything references by index.** Nodes reference roads and segments; segments reference nodes; the cost
  grids reference the graph. Change one array and every cross-reference must follow
  ([C18.2](02-node-record.md), [C18.3](03-segment-record.md)).
- **Global effects.** A single wrong segment cost or node position can distort routing far away — cops taking
  absurd paths, GPS lines through walls — because the search is global.
- **The frontier bites hardest on writes.** You can *read* the graph without the cost grids; you cannot safely
  *rewrite* routing without them, because you'd be changing a system whose cost inputs you don't fully model.

So the practical stance is: **read freely, edit conservatively.**

## What you can safely do

Within the decoded parts, useful work is possible:

- **Reconstruct and visualise** the road map from node positions and segments — draw the whole network, verify
  connectivity ([C18.4](04-graph.md)).
- **Analyse** intersections (node degree), road lengths (arc length), and topology.
- **Small, closure-preserving edits** — e.g. nudging a node position — *if* you keep every invariant true
  (ordinals contiguous, adjacency mutual, degrees correct) and accept that costs/flags you didn't touch still
  govern routing.

## What to avoid

- **Walking CARP with the chunk reader** ([C18.1](01-carp-format.md)) — corrupts on write.
- **Rewriting segment flags or cost grids** you haven't decoded — global routing breakage.
- **Adding/removing nodes or segments without fixing every cross-reference** — dangling indices, broken
  closure.
- **Assuming distance = cost** — the cost grids mean the cheapest route isn't always the shortest.

## Verifying any road edit

If you do edit within the safe envelope, verify with the closure checks ([C18.4](04-graph.md)):

1. every node ordinal equals its array position;
2. `Σ node degree == 2 × segmentCount`;
3. every segment endpoint is a valid node;
4. node segment-lists and segment endpoints agree.

Then drive it: traffic flows sanely, cops route plausibly, and the GPS line follows real roads. Because so much
is cross-referenced and partially decoded, the in-game test is not optional here — it is the only way to catch
a routing regression the file checks can't.

---

### Key takeaways

- Decoded: CARP container, nodes, segments, graph closure. Frontier: segment flags, `CGrd`/`CGcn` cost grids,
  `RNpf`/`RNrd` internals.
- The cost grids hold routing costs beyond arc length — they shape cop and GPS routes and are the least
  decoded.
- Editing is expert territory: everything cross-references by index, effects are global, and the frontier bites
  hardest on writes.
- Safe work: reconstruct/visualise the map, analyse topology, tiny closure-preserving tweaks.
- Verify with the four closure checks and, decisively, by driving — routing regressions hide from file checks.

**Continue:** [Chapter 19 — Audio: Banks (SNR/SPT/ABK)](../C19-Audio-Banks/C19-Audio-Banks.md) ·
[Chapter 18 hub](C18-Road-Network-CARP.md)
