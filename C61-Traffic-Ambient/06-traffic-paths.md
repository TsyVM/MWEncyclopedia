# C61.6 — The Track Path Network

> **The one-sentence version:** the AI/traffic routes are an on-disk path network in the master file — a
> `TrackPathManager` container (`0x80034150`) whose `0x34152` block is ~443 path *segments*, each a `{11,11}` record
> carrying an id, a 2-D bounding box and anchor, and a polyline of world-space `(x, y)` waypoints — decoded
> byte-for-byte against retail `L2RA.BUN`.

[← C61.5 — Reading traffic in RE](05-reading-traffic.md) · [Chapter 61 hub](C61-Traffic-Ambient.md) ·
[Next: Chapter 62 — Physics Constraints, Joints & Trailers →](../C62-Constraints-Joints/C62-Constraints-Joints.md)

---

## The routes traffic follows

The traffic runtime ([C61.1](01-traffic-system.md)–[C61.4](04-traffic-behavior.md)) populates the world with cars
that *drive the roads* — but *which* roads, along *what* lines? That comes from an on-disk **path network**: a
`TrackPathManager` container, chunk **`0x80034150`**, in the master track file
([C15.1](../C15-Track-Streaming/01-master-layout.md)). It holds four children:

| Child | Size (L2RA) | Role |
|---|---|---|
| `0x00034151` | 808 B | segment **index** (`{40, 373}` header, then id entries) |
| `0x00034152` | 52,876 B | the **path segments** — id + bbox + waypoint polyline ([below](#the-segment-record)) |
| `0x00034153` | 46,904 B | **connectivity** — `u16` index arrays linking segments |
| `0x00034155` | 2,964 B | **control-point config** (opens with the tag `CP1`) |

So the network is the classic *geometry + topology + index* split: `0x34152` is *where the paths go* (the
waypoints), `0x34153` is *how they connect* (the graph), `0x34151` indexes them, and `0x34155` configures control
points. This is the *route layer* the AI drives — related to, but distinct from, the `CARP` road graph
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) that the GPS routes on
([C29.5](../C29-Minimap-Map-Data/05-road-overlay.md)).

> ✅ *Verified:* `0x80034150` in retail `L2RA.BUN` contains `0x34151`/`0x34152`/`0x34153`/`0x34155`; `0x34152` is a
> dense array of world-space float pairs and `0x34155` opens with the ASCII tag `CP1` — read directly from the file.

## The segment record

The heart is `0x00034152` — the **path segments**. It's an array of variable-length records, each a path with its
own waypoint polyline:

```
per record:
  +0x00  u32  0x0B         version pair (11, 11)   — the world-data record convention
  +0x04  u32  0x0B
  +0x08  u16  pathId        (101, 102, 103, … — near-sequential segment ids)
  +0x0A  u16  waypointCount
  +0x0C  f32[4]  bbox       (minX, minY, maxX, maxY)   — the segment's 2-D bound
  +0x1C  f32[2]  anchor     (x, y)                     — a centre/entry point
  +0x24  waypointCount × { f32 x, f32 y }   — the path polyline (world, ground plane)
record size = 0x24 + waypointCount × 8
```

The `{11, 11}` version pair opens the record, the same convention scenery, smackable, and position-marker records use
([C63.9](../C63-Collision-World/09-smackables-emitters.md), [C17.7](../C17-Triggers-Barriers/07-markers-sequences.md))
— a shared world-data record header. Then an **id**, a **count**, a **2-D bounding box** and **anchor**, and the
**waypoint polyline**: `count` pairs of world `(x, y)` (height omitted — traffic follows the ground, like triggers
[C17.1](../C17-Triggers-Barriers/01-footprints.md)).

> ✅ *Verified against retail `L2RA.BUN`:* walking the records by `size = 0x24 + count·8` lands cleanly on
> **442/443** segment boundaries; the block holds **~4,136 waypoints** spanning **X ∈ [−2165, 7750]**, **Y ∈
> [−3440, 7097]** (Rockport world scale); and **94%** of each record's waypoints fall inside that record's *own*
> declared `bbox` (`+0x0C`) — a strong cross-check that the id/count/bbox/waypoint layout is correct.
> 🟡 *Reasoned:* the `anchor` (`+0x1C`) reads as a per-segment centre/entry point; the exact `0x34153` connectivity
> encoding and the `0x34155` `CP1` fields are partially decoded (index arrays / config). The `0x34152` record
> structure, the walk, and the in-bbox check are verified.

## Why a separate path network

Why store traffic routes *separately* from the road graph ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md))?
Because they answer different questions:

- **The `CARP` road graph** (`0x3B800`, [Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) — nodes and
  segments for *routing*: the GPS shortest path ([C29.5](../C29-Minimap-Map-Data/05-road-overlay.md)), where roads
  meet, arc-length costs. A *graph for pathfinding*.
- **The track path network** (`0x34152`, this page) — dense *waypoint polylines* for *following*: the exact line a
  traffic car (or a cop, [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)) drives down a
  stretch of road, lane by lane. A *geometry for driving*.

So the road graph is *coarse and topological* (for deciding a route), and the path network is *fine and geometric*
(for driving it smoothly). A traffic car ([C61.4](04-traffic-behavior.md)) is assigned a path segment and steers
along its waypoints; the AI driver ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)) follows the polyline,
swerving off it only to avoid the player. The two layers cooperate: the graph picks *which way*, the path network
provides *the exact line*. This is the on-disk complement to the traffic *runtime* — the runtime
([C61.1](01-traffic-system.md)–[C61.4](04-traffic-behavior.md)) is the *cars*, this network is the *roads they know
how to drive*.

## Editing the path network

The segment format is size-neutral-friendly for *reshaping* but a repack for *adding*:

- **Move a path** — rewrite its waypoint `(x, y)` pairs in place (same count → same size,
  [C75.2](../C75-Modding-Workflow/02-inplace-vs-repack.md)); recompute the `bbox` (`+0x0C`) to bound the new points,
  or the culling that uses it ([below](#the-segment-record)) rejects the segment wrongly.
- **Add/remove waypoints** — changes `waypointCount` and the record size, so `0x34152` grows and the `0x80034150`
  parent's size must be fixed ([C75.3](../C75-Modding-Workflow/03-ancestor-fixups.md)) — a repack. The connectivity
  (`0x34153`) and index (`0x34151`) reference segments, so keep them consistent
  ([C75.5](../C75-Modding-Workflow/05-distribution.md)).

So retiming or re-routing traffic is a path-network edit: move the waypoints (in place) to change the line, or
repack to change a route's length. The same size-neutral discipline the rest of the world data needs
([C15.7](../C15-Track-Streaming/07-section-contents.md)) applies here.

## RE implications

- **`0x80034150` `TrackPathManager`** — the on-disk AI/traffic route network: index (`0x34151`), segments
  (`0x34152`), connectivity (`0x34153`), config (`0x34155`).
- **`0x34152` segment** — `{11,11}` + `u16 id` + `u16 count` + `bbox` + `anchor` + `count × vec2` waypoints; size
  `0x24 + count·8`.
- **Graph vs geometry** — the `CARP` road graph ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md))
  *routes*; the path network *drives* (the exact waypoint line).
- **Editing** — reshape in place (recompute bbox); add waypoints = repack with ancestor-size fixups.

---

### Key takeaways

- The traffic/AI **routes** are an on-disk **path network** — chunk **`0x80034150` (`TrackPathManager`)** in the
  master file — holding an index (`0x34151`), the **segments** (`0x34152`), **connectivity** (`0x34153`), and a
  `CP1` config (`0x34155`).
- Each **`0x34152` segment** is a `{11,11}` record: **`u16 id`**, **`u16 waypointCount`**, a **2-D `bbox`** and
  **anchor**, then **`count × (x, y)`** world-space waypoints — record size **`0x24 + count·8`** — decoded
  byte-for-byte against retail `L2RA.BUN` (442/443 clean segment walks, **94%** of waypoints inside their own bbox).
- It's **geometry, not a graph** — the dense waypoint polyline a car *drives*, complementing the coarse `CARP` road
  graph ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) that *routes*: the graph picks the way, the
  path network gives the exact line the AI follows ([C47.4](../C47-AI-Driver-Vehicle/04-navigation-systems.md)).
- This is the **on-disk complement** to the traffic runtime — the runtime
  ([C61.1](01-traffic-system.md)–[C61.4](04-traffic-behavior.md)) is the cars; this network is **the roads they know
  how to drive**.
- **Editing**: reshape a path by rewriting its waypoints in place (recompute the bbox); adding waypoints is a
  **repack** with ancestor-size fixups ([Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md)).

**Continue:** [Chapter 62 — Physics Constraints, Joints & Trailers](../C62-Constraints-Joints/C62-Constraints-Joints.md) ·
[Chapter 61 hub](C61-Traffic-Ambient.md)
