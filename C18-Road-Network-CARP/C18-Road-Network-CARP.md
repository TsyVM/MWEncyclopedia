# Chapter 18 — The Road Network (CARP)

> **Goal of this chapter:** decode the graph the game's AI and GPS drive on — the `CARP` road network of nodes
> and segments — and understand why it is *attribute data, not a chunk tree*, how the graph closes, and how
> cops, traffic, and the GPS line route over it.

Every car in Most Wanted that isn't you — the traffic, the police, the rivals — and the GPS line that guides
you all navigate the same structure: the **CARP road network**, a graph of **nodes** (intersections and road
points) joined by **segments** (the road between them). This chapter decodes that graph from the retail track
and shows how routing rides on it.

> **Verified against retail data.** The road network is `0x0003B800 WorldMapData` in `TRACKS/L2RA.BUN`, and
> this book confirmed directly: the payload carries the **`CARP`** magic (stored reversed as `PRAC`) and a
> directory of tags — `RNnd`, `RNsg`, `RNrd`, `RNpf`, `RNgp`, `RNhd`, `CGrd`, `CGcn`. The directory gives
> **4,385 nodes** (`RNnd`, `0x1121`), **6,538 segments** (`RNsg`, `0x198A`), and 1,308 roads (`RNrd`); node
> records carry the documented `0xAAAA`-padded segment list. The node/segment field layouts were decoded
> against the full graph with the adjacency closing exactly (Σ node degree = 2 × segments).

---

## Deep-dive pages

- [C18.1 — CARP is a TLV blob, not a chunk tree](01-carp-format.md): the critical structural fact, the tag
  directory, and why the universal chunk reader must branch around it.
- [C18.2 — The RNnd node](02-node-record.md): the 32-byte node — position, ordinal, road index, degree, and
  the segment list.
- [C18.3 — The RNsg segment](03-segment-record.md): the 22-byte segment — its two endpoint nodes, arc length,
  and flags.
- [C18.4 — The graph: adjacency & roads](04-graph.md): how nodes and segments close into a navigable graph,
  and what `RNrd` roads add.
- [C18.5 — Routing: cops, traffic & GPS](05-routing.md): how the AI and the GPS line query the graph.
- [C18.6 — The frontier: cost grids & editing](06-frontier-editing.md): the `CGrd`/`CGcn` cost grids, what's
  still open, and why editing the graph is expert territory.

---

## 18.1 CARP is not an EAGL chunk tree

The single most important fact — the one that trips up every tool — is that **`CARP` is attribute data, not a
nested `{id, size}` chunk tree** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)). Inside
the `0x0003B800` chunk, the payload is a `CARP` TLV blob with its own directory of four-character tags
(`RNnd`, `RNsg`, …), each pointing at a count and a data offset. Feed it to the universal chunk walker and it
will read `RNnd`/`RNsg` as chunk headers, "parse" garbage, and — if you write it back — corrupt the graph. A
correct reader **branches `CARP` out** of the chunk walker and parses it as a tag directory
([C18.1](01-carp-format.md)).

## 18.2 Nodes are 32-byte graph vertices

`RNnd` is the node array — 4,385 of them in the worked track. Each **32-byte** node is a graph vertex: a
world **position** `(x, h, y)`, its **ordinal** (`nodeIndex`, equal to its array position), a **road index**
(`→ RNrd`), a **degree** (how many segments meet here), and a **segment list** of up to seven neighbour
segment indices, `0xAAAA`-padded to fill the record ([C18.2](02-node-record.md)). Verified: the node records
carry exactly that `0xAAAA` padding, and the ordinals run 0…4,384.

## 18.3 Segments are 22-byte edges

`RNsg` is the segment array — 6,538 edges. Each **22-byte** segment joins **two endpoint nodes** (`nodeA`,
`nodeB`) and carries the road geometry between them — an **arc length** (stored scaled) and flags
([C18.3](03-segment-record.md)). Segments are the graph's edges; nodes are its vertices; together they are a
navigable road graph.

## 18.4 The graph closes exactly

The proof that the node/segment decode is correct is that the graph **closes**: every segment names two valid
nodes, and the sum of all node degrees equals twice the segment count (each segment contributes to two nodes'
lists). Verified on the retail graph — the adjacency is exact, no dangling edges, no orphan nodes
([C18.4](04-graph.md)). `RNrd` roads group segments into named roads (streets), the layer the GPS labels and
the map draws.

## 18.5 Everything AI drives on this graph

The road network is the substrate for all non-player navigation:

- **Traffic** flows along segments between nodes, picking turns at intersections.
- **Cops** pathfind over the graph to intercept and pursue you
  ([Chapter 14](../C14-Vault-Pursuit-Surfaces/01-pursuit-ai.md)).
- **The GPS line** is a shortest-path query over the graph's cost data, drawn as the route to your objective.

All three ask the same graph "how do I get from here to there," differing only in cost function and goal
([C18.5](05-routing.md)).

## 18.6 The cost grids are the frontier

Routing needs *costs*, and those live in the `CGrd`/`CGcn` **cost grid** tags — the part of CARP that is least
decoded. Node and segment structure is solid (this chapter), but the cost grid layout, the segment flag
semantics, and `RNpf`/`RNrd` internals remain partial, which is why *rewriting* the graph is expert territory:
you can read and reason about it, but editing routing safely requires the pieces still on the frontier
([C18.6](06-frontier-editing.md)).

---

### Key takeaways

- The road network is `0x0003B800` — a **`CARP` TLV blob**, *not* a chunk tree; branch it out of the universal
  reader.
- Verified directory: **4,385 `RNnd` nodes (32 B)**, **6,538 `RNsg` segments (22 B)**, 1,308 `RNrd` roads, plus
  `CGrd`/`CGcn` cost grids.
- Nodes are graph vertices (position, ordinal, road index, degree, `0xAAAA`-padded segment list); segments are
  edges (two endpoint nodes, arc length, flags).
- The graph closes exactly (Σ degree = 2 × segments) — the proof the decode is right.
- Traffic, cops, and the GPS line all route on this one graph; the cost grids that drive routing are the
  decode frontier.

**Next:** [Chapter 19 — Audio: Banks (SNR/SPT/ABK)](../C19-Audio-Banks/C19-Audio-Banks.md): leaving the world
for the game's sound.
