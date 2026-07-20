# C29.6 — Editing the Map

> **The one-sentence version:** re-skin the map by editing the tile TPKs (JDLZ → TPK → JDLZ) across all zoom
> levels, keep the projection metadata matched to the world, and remember overlays and the GPS route come from
> other systems — not the map imagery.

[← C29.5 — The road network on the map](05-road-overlay.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md) ·
[Next: Chapter 30 — Localization: String Tables & the Label System →](../C30-Localization-Labels/C30-Localization-Labels.md)

---

## What lives where

Before editing, know which piece owns what, because the map draws from several systems:

| To change… | Edit… |
|---|---|
| Map **imagery** (the picture) | the tile TPKs ([C29.1](01-tiles.md)) + `TrackMaps.bin` textures ([C29.2](02-trackmaps.md)) |
| Icon **art** (player, cops, flags) | the HUD/UI atlases ([C29.4](04-overlays.md), [C27.2](../C27-FrontEnd-Shell-UI/02-ui-atlases.md)) |
| Where entities **appear** | the world→map projection ([C29.3](03-world-to-map.md)) |
| The **GPS route** | the road network ([C18.6](../C18-Road-Network-CARP/06-frontier-editing.md)) |
| Road **names** | the `RNrd` grouping ([C18.4](../C18-Road-Network-CARP/04-graph.md)) |

So "editing the map" is usually **re-skinning the tiles** — the other elements are computed from systems
decoded in their own chapters.

## Re-skinning the tiles

The common map edit is changing the map's look — a restyled map, a custom overlay style:

```python
def reskin_tile(tile_file):
    chunk = read_chunk(tile_file)                # 0x0003A100
    tpk = jdlz_decompress(chunk.payload)         # C29.1
    edit_tpk_textures(tpk)                        # re-skin the map image (C5.5)
    chunk.payload = jdlz_compress(tpk)           # recompress (C3)
    write_chunk(tile_file, chunk)
```

The rules combine the JDLZ, TPK, and tile constraints:

- **JDLZ round-trip** — decompress, edit, recompress ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)).
- **TPK discipline** — same-size/format texture edits are cleanest ([C5.5](../C5-Textures-TPK/05-extract-replace.md),
  [Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).
- **All zoom levels** — edit every `MINI_MAP_*` variant for the region, or the map looks different at different
  zooms ([C29.1](01-tiles.md)).
- **Keep the grid** — don't change the tile/zoom scheme, or the projection ([C29.3](03-world-to-map.md)) and
  loading break.

## Don't move the projection

The world→map projection ([C29.3](03-world-to-map.md)) is the contract between the map imagery and the world.
Change the tile *art* freely, but don't change the *projection* (origin/scale) unless you deliberately re-grid
the map — because every overlay ([C29.4](04-overlays.md)) and the GPS line ([C29.5](05-road-overlay.md)) project
through it. A mismatched projection puts the player marker in the wrong place and the GPS line off the roads,
even with perfect tile art.

## Overlays and routes aren't map edits

A frequent confusion: to change what the map *shows about gameplay*, you edit the gameplay systems, not the
map:

- **Cop visibility** → pursuit/heat ([Chapter 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)).
- **The route** → the road network ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)).
- **Objective markers** → the POI/event systems ([plan Chapter 55](../README.md)).

The map is a *display* of these systems ([C29.5](05-road-overlay.md)); it draws what they compute. So re-skin
the map for looks; edit the systems for behaviour.

## Verify

After a map edit:

1. **Tiles decompress and parse** — JDLZ → TPK → texture, at every zoom
   ([C29.1](01-tiles.md)).
2. **The projection is unchanged** (unless intended) — overlays still align
   ([C29.3](03-world-to-map.md)).
3. **In game** — open the map and drive: the imagery looks right at all zooms, the player marker sits where the
   car is, and the GPS line follows roads ([C29.5](05-road-overlay.md)).

The in-game map check catches the two classic map bugs — inconsistent zoom levels and a misaligned projection —
that file checks alone miss.

---

### Key takeaways

- "Editing the map" is mostly **re-skinning the tiles** — icons, routes, and reveals come from other systems.
- Re-skin tiles with the JDLZ→TPK→JDLZ round-trip, editing **all zoom levels**, same format/size, keeping the
  grid.
- **Never move the projection** (origin/scale) unless re-gridding — it aligns all overlays and the GPS line.
- Change gameplay-on-the-map (cop reveal, route, objectives) in the **gameplay systems**, not the map imagery.
- Verify tiles at every zoom, projection alignment, and the in-game map (marker position, GPS on roads).

**Continue:** [Chapter 30 — Localization: String Tables & the Label System](../C30-Localization-Labels/C30-Localization-Labels.md) ·
[Chapter 29 hub](C29-Minimap-Map-Data.md)
