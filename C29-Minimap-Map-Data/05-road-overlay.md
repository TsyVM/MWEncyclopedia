# C29.5 — The Road Network on the Map

> **The one-sentence version:** the GPS line is a shortest-path query over the CARP road graph from the player
> to the objective, projected to map space and drawn over the tiles — so the map's route is the same road
> network the AI drives on.

[← C29.4 — Overlays: player, objectives, cops](04-overlays.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md) ·
[Next: C29.6 — Editing the map →](06-editing-map.md)

---

## The GPS line is a road-graph query

The route the map draws to your objective — the GPS line — is not painted onto the tiles; it is **computed** as
a **shortest path over the CARP road network** ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md))
from the player's position to the objective, then projected to map space ([C29.3](03-world-to-map.md)) and drawn
as a line over the map:

```
player position, objective position
   → shortest path over the CARP graph (nodes + segments, arc-length/cost — C18)
   → a sequence of road segments
   → project each to map pixels (C29.3)
   → draw the connected line over the tiles
```

So the GPS line follows **real roads** — because it's a path over the actual road graph the traffic and cops
route on ([C18.5](../C18-Road-Network-CARP/05-routing.md)). Take a wrong turn and it re-queries from your new
position, exactly as the routing chapter describes.

## The map ties decoded systems together

The road overlay is where several decoded systems converge on one display:

- **Tiles** ([C29.1](01-tiles.md)) — the map imagery.
- **Projection** ([C29.3](03-world-to-map.md)) — world → map pixels.
- **Overlays** ([C29.4](04-overlays.md)) — player, cops, objectives.
- **Road network** ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) — the GPS route.

The map is a small integration point: it draws the world (tiles), locates entities in it (projection +
overlays), and routes through it (road graph). Understanding the map means understanding how these pieces fit —
which is why it comes after the world, streaming, and road-network chapters.

## Roads and labels

Beyond the GPS line, the map can label **roads** by name — the `RNrd` road grouping
([C18.4](../C18-Road-Network-CARP/04-graph.md)) provides the named streets, so the map (and GPS) can say "turn
onto Rosewood Drive." So the road network serves the map twice: the **fine graph** (nodes + segments) computes
the route, and the **road grouping** (`RNrd`) labels it. The tiles show the roads as imagery; the graph adds the
route and the names.

> ✅ *Verified (via Chapter 18):* the road network is the CARP graph (`RNnd`/`RNsg`/`RNrd`), decoded in
> [Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md); the GPS line is a shortest-path query over it
> ([C18.5](../C18-Road-Network-CARP/05-routing.md)).
> 🟡 *Reasoned:* the specific projection of the path onto the map and the line-drawing are the map's rendering
> of the verified graph query.

## The GPS re-queries as you drive

Like the cop/AI routing ([C18.5](../C18-Road-Network-CARP/05-routing.md)), the GPS line is **live**: as you
move, the path is recomputed from your current node so it always leads from where you are. Miss a turn and the
line reroutes; near the objective it shortens. So the road overlay, like the entity overlays
([C29.4](04-overlays.md)), is a per-frame (or per-move) recomputation — the map's route is always current.

## Editing implications

- **The route follows the road graph** — to change routing you edit the road network
  ([C18.6](../C18-Road-Network-CARP/06-frontier-editing.md)), not the map; the map just draws the query.
- **Re-skinning tiles doesn't change routes** — the line is computed, not painted
  ([C29.1](01-tiles.md)).
- **Road labels come from `RNrd`** ([C18.4](../C18-Road-Network-CARP/04-graph.md)) — renaming a road is a
  road-network edit.
- **Keep the projection consistent** — the line only follows roads if world→map ([C29.3](03-world-to-map.md))
  matches the world the graph is in.

---

### Key takeaways

- The **GPS line** is a shortest-path query over the CARP road graph (Chapter 18), projected and drawn over the
  tiles.
- It follows **real roads** because it's a path over the same graph the AI drives on.
- The map integrates decoded systems: tiles (imagery), projection (world→map), overlays (entities), road network
  (route).
- Roads are labelled from the `RNrd` grouping; the graph both **computes** and **names** the route.
- The route is **live** (re-queried as you move); change routing via the road network, not the map.

**Continue:** [C29.6 — Editing the map](06-editing-map.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md)
