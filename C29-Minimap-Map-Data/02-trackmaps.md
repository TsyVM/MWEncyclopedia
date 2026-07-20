# C29.2 — TrackMaps.bin

> **The one-sentence version:** `TrackMaps.bin` is the map's EAGL container (`0xB3300000`) — the organised store
> of map textures and data that backs the minimap and full map, the map's counterpart to the world's master
> track file.

[← C29.1 — The map tiles](01-tiles.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md) ·
[Next: C29.3 — Positioning: world → map →](03-world-to-map.md)

---

## The map's container

`TRACKS/L2RA/TrackMaps.bin` is an **EAGL container** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md),
verified magic `0xB3300000`) that holds the map's data — textures ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md))
and the metadata that organises them. Where the individual `MINI_MAP*.BIN` tiles ([C29.1](01-tiles.md)) are the
streamed pieces, `TrackMaps.bin` is the container/index that ties the map together for a track.

It plays the same role for the *map* that the master track file
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) plays for the *world*: an organised store of
imagery plus the data that positions and indexes it.

## What it holds

As an EAGL container, `TrackMaps.bin` walks like any chunk file
([C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md)):

- **Map textures** — TPKs of map imagery ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)).
- **Map metadata** — the projection/positioning data that maps world coordinates to map space
  ([C29.3](03-world-to-map.md)) and organises the tiles/zoom levels ([C29.1](01-tiles.md)).

So decoding the map is: read `TrackMaps.bin` (and the tiles) as EAGL/TPK, and interpret the metadata as the
map's world→pixel transform and tile organisation.

> ✅ *Verified:* `TrackMaps.bin` is an EAGL container (magic `0xB3300000`); it is the map's texture/data store,
> parallel to the world master file.
> 🟡 *Reasoned:* the precise internal chunk breakdown (which chunk is projection vs tile index) is the map
> format's detail; its EAGL/TPK identity and role are verified.

## Per-track maps

The file lives under a track directory (`TRACKS/L2RA/`), so **each track has its map data**. Most Wanted's
open world is one large track (L2RA — the Rockport map), so `TrackMaps.bin` there is the whole-city map. A
game with multiple discrete tracks would have a `TrackMaps.bin` per track; here it's the one open-world map. The
per-track organisation mirrors the world files ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)),
which are also per-track (`L2RA.BUN`, `STREAML2RA.BUN`).

## The map and the world share coordinates

Because `TrackMaps.bin` sits beside the world files for the same track, the map and the world share a
coordinate system — the map's projection ([C29.3](03-world-to-map.md)) is defined in the same world space the
geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), streaming grid
([C15.4](../C15-Track-Streaming/04-world-grid.md)), and road network
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) use. That shared frame is what lets the map
overlay live world positions (the player, cops, the GPS line — [C29.4](04-overlays.md)–[C29.5](05-road-overlay.md))
accurately: everything is in the same world coordinates, projected the same way.

## Editing implications

- **Read it as EAGL/TPK** — the universal tools ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md),
  [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) apply.
- **Re-skin map textures** within it like any TPK ([C5.5](../C5-Textures-TPK/05-extract-replace.md)).
- **Don't disturb the projection metadata** unless you intend to remap world→map — it must match the world
  coordinates ([C29.3](03-world-to-map.md)).
- **Keep it consistent with the tiles** — `TrackMaps.bin` and the `MINI_MAP*` tiles are one map system; edit
  them coherently.

---

### Key takeaways

- `TrackMaps.bin` is the map's **EAGL container** (`0xB3300000`) — textures + metadata backing the map.
- It's the map's counterpart to the world master file: an organised store of imagery and positioning data.
- The map is **per-track**; for the open world it's the whole-city map, beside the world files.
- The map shares the **world coordinate system**, which is what lets overlays project accurately.
- Read/edit it as EAGL/TPK; preserve the projection metadata; keep it consistent with the tiles.

**Continue:** [C29.3 — Positioning: world → map](03-world-to-map.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md)
