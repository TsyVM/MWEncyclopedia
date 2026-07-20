# C18.5 — Routing: Cops, Traffic & GPS

> **The one-sentence version:** traffic, police, and the GPS line all navigate the *same* road graph — they
> differ only in cost function and goal — with the runtime loader handing the CARP payload to
> `WRoadNetwork::ParseCARP` to build the graph the AI queries.

[← C18.4 — The graph: adjacency & roads](04-graph.md) · [Chapter 18 hub](C18-Road-Network-CARP.md) ·
[Next: C18.6 — The frontier: cost grids & editing →](06-frontier-editing.md)

---

## One graph, three clients

Everything that navigates the world rides the road graph ([C18.4](04-graph.md)), asking the same fundamental
question — "how do I get from here to there?" — with different costs and goals:

| Client | Goal | Cost emphasis |
|---|---|---|
| **Traffic** | flow naturally along roads | follow segments, obey road sense; no destination |
| **Cops** | intercept / pursue the player | shortest time to the player, aggressive routing ([C14.1](../C14-Vault-Pursuit-Surfaces/01-pursuit-ai.md)) |
| **GPS line** | guide the player to an objective | shortest path to the goal, drawn on screen |

The graph is neutral; each client supplies its own cost function and destination and runs a search. This is why
one data structure serves the whole game's navigation.

## The runtime path

At load, the road network flows from disk to a live graph:

```
0x0003B800 (CARP blob)
  → loader (0x78A120) reads the WorldMapData chunk
  → WRoadNetwork::ParseCARP (0x789BE0) parses the tag directory + node/segment arrays
  → a live road graph the AI and GPS query each frame
```

`WRoadNetwork::ParseCARP` is the runtime counterpart of the parser in
[C18.1](01-carp-format.md) — it reads the same directory, node, and segment structures this chapter decodes and
builds the in-memory graph. So the on-disk format and the runtime graph are two views of the same data, which
is why decoding the file tells you how the AI sees the world.

## Traffic: flowing the graph

Traffic doesn't have a destination — it *flows*. A traffic car follows its current segment to a node, picks an
outgoing segment (respecting road sense), and continues, producing the ambient city traffic. The graph's
segments are the lanes; the nodes are the decision points (intersections). Density and behaviour are tuned by
the vault ([Chapter 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)), but the *paths* are the
road graph.

## Cops: pursuit routing

Police pathfind over the graph to reach *you* ([C14.1](../C14-Vault-Pursuit-Surfaces/01-pursuit-ai.md)) — a
shortest-path search from the cop's node to the player's, with a cost function biased toward speed and
interception (cutting you off, not just following). Roadblocks and reinforcements are placed at nodes ahead of
you on the graph. The pursuit's *aggression* is vault-tuned; its *routes* are graph searches. This is the
graph's most dynamic client — the search re-runs as you move.

## GPS: shortest path, drawn

The GPS line is a **shortest-path query** over the graph from your position to your objective, using arc-length
([C18.3](03-segment-record.md)) and the cost grids ([C18.6](06-frontier-editing.md)) as edge costs. The
resulting path of segments is drawn as the on-screen route and labelled by `RNrd` road names
([C18.4](04-graph.md)). When you take a wrong turn, the GPS re-queries from your new node — the same search,
re-run.

## The shared search

All three clients use the same underlying graph search (a shortest-path/A\* family algorithm over nodes and
segments), differing in:

- **Start and goal** — cop node → player node; player node → objective; traffic has no goal (local choice).
- **Cost function** — time vs distance vs "avoid the player" vs "follow road sense."
- **Re-planning cadence** — GPS on wrong turns, cops continuously, traffic per-intersection.

Understanding this shared substrate is the payoff of decoding the graph: the police that hound you and the line
that guides you are the same data, queried differently.

> ✅ *Verified:* the runtime loader (`0x78A120`) hands the CARP payload to `WRoadNetwork::ParseCARP`
> (`0x789BE0`), which parses the node/segment structures this chapter decodes.
> 🟡 *Reasoned:* the specific cost functions per client are described from behaviour and the graph's cost data;
> the graph and parse path are verified.

---

### Key takeaways

- Traffic, cops, and the GPS line all navigate the **same** road graph, differing only in cost function and
  goal.
- The loader (`0x78A120`) → `WRoadNetwork::ParseCARP` (`0x789BE0`) builds the runtime graph from the CARP
  structures this chapter decodes.
- Traffic flows the graph (no destination); cops shortest-path to the player; GPS shortest-paths to the
  objective and draws it.
- All use a shared graph search, varying start/goal, cost, and re-plan cadence.
- Decoding the file reveals how the AI sees the world — the on-disk graph *is* the runtime graph.

**Continue:** [C18.6 — The frontier: cost grids & editing](06-frontier-editing.md) · [Chapter 18 hub](C18-Road-Network-CARP.md)
