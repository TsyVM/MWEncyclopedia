# C29.1 — The Map Tiles

> **The one-sentence version:** the map imagery is a set of `MINI_MAP*.BIN` tiles — each a chunk (`0x0003A100`)
> wrapping a JDLZ-compressed TPK of the map picture — so the map streams and zooms by loading only the tiles a
> view needs.

[← Chapter 29 hub](C29-Minimap-Map-Data.md) · [Next: C29.2 — TrackMaps.bin →](02-trackmaps.md)

---

## Tiles cover the map

The world map is stored not as one giant image but as **tiles** — `MINI_MAP.BIN`, `MINI_MAP_10_2_1.BIN`,
`MINI_MAP_10_2_2.BIN`, `MINI_MAP_10_3_1.BIN`, … The naming encodes a **zoom/region grid** (`10_2_1` = zoom
level 10, grid cell 2, sub-tile 1, and so on): the map exists at multiple zoom levels, each subdivided into
tiles. A view loads the tiles for its region and zoom, not the whole map.

This is the 2-D image analogue of world streaming ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)):
just as the world is sectioned so only nearby geometry is resident, the map is tiled so only the visible
region/zoom is loaded. Zooming the map swaps to a different zoom level's tiles; panning loads adjacent tiles.

## Each tile is a JDLZ-wrapped TPK

A tile file, verified on `MINI_MAP.BIN`, is:

```
+0x00  0x0003A100   chunk id
+0x04  size
+0x08  "JDLZ" …     JDLZ-compressed payload (Chapter 3)
         └── (decompressed) a TPK of the tile's map image (Chapter 5)
```

So reading a tile is: parse the chunk, **JDLZ-decompress** the payload
([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)), then read the **TPK**
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) inside to get the tile's texture — decodable with the
codec chapter ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)). The map picture is thus ordinary
textures, compressed for footprint and chunk-wrapped, exactly the JDLZ+TPK combination seen elsewhere.

```python
def read_map_tile(tile_file):
    chunk = read_chunk(tile_file)            # 0x0003A100
    tpk_bytes = jdlz_decompress(chunk.payload)   # JDLZ at +8 (C3)
    return read_tpk(tpk_bytes)               # the tile's map texture (C5)
```

> ✅ *Verified:* `MINI_MAP.BIN` is a `0x0003A100` chunk with `JDLZ` at the payload — a JDLZ-compressed TPK map
> tile; the file set spans zoom/region variants (`MINI_MAP_10_2_1`, etc.).

## Why tile the map

Tiling the map at multiple zooms buys the same things tiling always does:

- **Streamable.** Load the visible tiles, not the whole map — a full-resolution map of the city is large.
- **Zoomable.** Pre-rendered zoom levels mean zooming swaps to appropriately-detailed tiles rather than scaling
  one image (which would blur or cost memory).
- **Compressible per tile.** Each tile is independently JDLZ+DXT compressed, so the map's footprint is small.

It's the map's version of the world's LOD/streaming trade: pre-authored detail levels, loaded on demand.

## Editing implications

- **Re-skin by editing the tile TPKs** — decompress JDLZ, edit the TPK ([C5.5](../C5-Textures-TPK/05-extract-replace.md)),
  recompress JDLZ ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)).
- **Keep the tile grid** — edit tiles in place; don't change the zoom/region scheme, or the map's projection
  ([C29.3](03-world-to-map.md)) and tile loading break.
- **Match format/dimensions** — same-size TPK swaps are cleanest ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).
- **All zoom levels** — a re-skin should touch every zoom level's tiles for consistency, or the map looks
  different at different zooms.

---

### Key takeaways

- The map is **tiled** (`MINI_MAP*.BIN`) at multiple zoom levels/regions — loaded on demand like world sections.
- Each tile is a `0x0003A100` chunk wrapping a **JDLZ-compressed TPK** (verified) — decompress then read the
  texture.
- Tiling gives streamable, zoomable, per-tile-compressible map imagery.
- Re-skin by editing the tile TPKs (JDLZ → TPK → JDLZ), keeping the grid and format.
- Touch all zoom levels for a consistent re-skin.

**Continue:** [C29.2 — TrackMaps.bin](02-trackmaps.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md)
