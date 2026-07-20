# Chapter 29 — Minimap: Tiles & Map Data

> **Goal of this chapter:** decode the minimap — JDLZ-wrapped TPK tiles of the map imagery plus the map data
> that positions and overlays them — so you can read and re-skin the in-game map and understand how the player,
> objectives, and cops are drawn on it.

The minimap in the HUD ([C27.4](../C27-FrontEnd-Shell-UI/04-hud.md)) and the full-screen map are the player's
spatial reference — where they are, where to go, where the cops are. Mechanically it's the **atlas + table**
pattern once more: **tiles** (map imagery) plus **map data** (positioning and overlay). This chapter decodes
both.

> **Grounded in verified formats.** The minimap tiles ship as `TRACKS/L2RA/MINI_MAP*.BIN` — each a chunk
> (`0x0003A100`) wrapping **JDLZ**-compressed ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) TPK
> tile data (verified `JDLZ` magic at the chunk payload). `TRACKS/L2RA/TrackMaps.bin` is an EAGL container
> (magic `0xB3300000`) — the map's texture/data. The map overlays the player and icons over these tiles.

---

## Deep-dive pages

- [C29.1 — The map tiles](01-tiles.md): the JDLZ-wrapped TPK tiles and how they cover the map.
- [C29.2 — TrackMaps.bin](02-trackmaps.md): the map's container and data.
- [C29.3 — Positioning: world → map](03-world-to-map.md): mapping world coordinates onto the map.
- [C29.4 — Overlays: player, objectives, cops](04-overlays.md): the icons drawn over the tiles.
- [C29.5 — The road network on the map](05-road-overlay.md): drawing the GPS line and roads.
- [C29.6 — Editing the map](06-editing-map.md): re-skinning tiles and adjusting overlays.

---

## 29.1 The map is tiles

The map imagery is stored as **tiles** — `MINI_MAP.BIN`, `MINI_MAP_10_2_1.BIN`, `MINI_MAP_10_2_2.BIN`, … — a
set of images that together cover the world map at various zoom levels / regions. Each tile is a chunk
(`0x0003A100`) wrapping a **JDLZ-compressed** ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) TPK
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) of the map picture ([C29.1](01-tiles.md)). Tiling lets the
map stream and zoom: load the tiles for the visible region/zoom, not the whole map at full resolution — the
same logic as world streaming ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) applied to a 2-D
image.

## 29.2 TrackMaps.bin

`TrackMaps.bin` is the map's EAGL container ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md),
magic `0xB3300000`) — holding the map textures and data that back the minimap and full map
([C29.2](02-trackmaps.md)). It's the map's counterpart to the world's master track file
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)): the organised store of map imagery and the
metadata that places it.

## 29.3 World → map

Drawing the map requires mapping **world coordinates** (Z-up, [C8.4](../C8-Geometry-Solids/04-bounding-boxes.md))
onto **map coordinates** (the 2-D tile space). Because the world is Z-up, the map is the horizontal (X-Y)
projection — the top-down footprint, the same plane triggers ([C17.1](../C17-Triggers-Barriers/01-footprints.md))
and the streaming grid ([C15.4](../C15-Track-Streaming/04-world-grid.md)) live in. A world position projects to
a tile pixel via a scale/offset transform, which is how the player's world location becomes a dot on the map
([C29.3](03-world-to-map.md)).

## 29.4 Overlays

Over the static tiles, the map draws **live overlays** ([C29.4](04-overlays.md)):

- **The player** — a marker at the player's projected position, oriented to heading.
- **Objectives** — race start/finish, event markers, POIs ([plan Chapter 55](../README.md)).
- **Cops** — pursuit vehicles ([Chapter 14](../C14-Vault-Pursuit-Surfaces/01-pursuit-ai.md)) shown during a
  chase.
- **The GPS line** — the route to the objective ([C29.5](05-road-overlay.md)).

These are HUD-style elements ([C27.4](../C27-FrontEnd-Shell-UI/04-hud.md)) positioned by world→map projection,
updated each frame — the *dynamic* layer over the *static* tiles.

## 29.5 The road network on the map

The **GPS line** is a road-network query drawn on the map: a shortest path over the CARP graph
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) from the player to the objective, projected to
map space and drawn as a route ([C29.5](05-road-overlay.md)). So the map ties together the tiles (imagery), the
projection (world→map), the overlays (live markers), and the road network (the route) — several decoded systems
converging on one display.

---

### Key takeaways

- The minimap is the **atlas + table** pattern: **tiles** (JDLZ-wrapped TPK map imagery) + **map data**
  (positioning/overlay).
- Tiles (`MINI_MAP*.BIN`, chunk `0x0003A100` + JDLZ) cover the map at zoom levels/regions, streamed like the
  world.
- `TrackMaps.bin` (EAGL, `0xB3300000`) is the map's container of textures and data.
- World (Z-up) → map is the horizontal projection; a world position becomes a map pixel via scale/offset.
- **Overlays** (player, objectives, cops, GPS line) are the live layer; the GPS line is a CARP road-graph query
  (Chapter 18).

**Next:** [Chapter 30 — Localization: String Tables & the Label System](../C30-Localization-Labels/C30-Localization-Labels.md):
the text the UI and map show.
