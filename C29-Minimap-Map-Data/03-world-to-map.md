# C29.3 — Positioning: World → Map

> **The one-sentence version:** the map is the horizontal (X-Y) projection of the Z-up world, so a world
> position becomes a map pixel through a scale/offset transform — the same transform that places the player,
> cops, and GPS line on the map.

[← C29.2 — TrackMaps.bin](02-trackmaps.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md) ·
[Next: C29.4 — Overlays: player, objectives, cops →](04-overlays.md)

---

## The map is the ground plane

Because the world is **Z-up** ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)), the map is its **horizontal
projection**: drop the Z (height) and you have the top-down X-Y footprint that the map depicts. This is the
same plane the trigger footprints ([C17.1](../C17-Triggers-Barriers/01-footprints.md)) and the streaming grid
([C15.4](../C15-Track-Streaming/04-world-grid.md)) live in — the map is Rockport seen from directly above.

So mapping a world position to the map discards height and transforms the (X, Y) horizontal coordinates into
map/tile space.

## The scale/offset transform

World→map is an affine 2-D transform — a scale and an offset (and possibly a rotation) taking world (X, Y) to
map pixels:

```
map_x = (world_x - origin_x) * scale_x
map_y = (world_y - origin_y) * scale_y
```

- **origin** shifts the world's coordinate range to the map's (so the map's corner is at map pixel 0,0).
- **scale** converts world units (metres) to map pixels (setting the zoom).

This transform — its origin and scale — is part of the map metadata ([C29.2](02-trackmaps.md)). Applying it to
the player's world position gives the pixel to draw the player marker ([C29.4](04-overlays.md)); applying it to
any world entity places it on the map.

> 🟡 *Reasoned:* the world→map projection as a Z-drop + scale/offset is the standard top-down-map transform,
> consistent with the verified Z-up world ([C8.4](../C8-Geometry-Solids/04-bounding-boxes.md)) and the map's
> shared coordinate system ([C29.2](02-trackmaps.md)); the exact origin/scale constants are in the map
> metadata.

## Zoom is a scale change

Because the map exists at multiple zoom levels ([C29.1](01-tiles.md)), zooming is changing the **scale** of the
world→map transform (and selecting the matching tiles):

- **Zoomed out** — small scale, the whole city fits, coarse tiles.
- **Zoomed in** — large scale, a neighbourhood fills the view, detailed tiles.

The player stays centered by adjusting the **offset** so the player's world position maps to the map's center.
So panning and zooming the map are just offset and scale changes to the same projection — the tiles
([C29.1](01-tiles.md)) provide the imagery at the right detail for the current scale.

## Why the shared coordinate frame matters

The map's accuracy depends entirely on using the **same world coordinates** as everything else
([C29.2](02-trackmaps.md)):

- The **player marker** is the player's world position projected — so it's exactly where the car is.
- **Cop icons** ([C29.4](04-overlays.md)) are cops' world positions projected — accurate pursuit tracking.
- **The GPS line** ([C29.5](05-road-overlay.md)) is a road-network path in world coordinates, projected — so it
  follows real roads.

One projection, applied to every world entity, keeps the whole overlay consistent with the world. This is why
the map "just works" — it's a single transform over the shared coordinate frame.

## Editing implications

- **Don't break the projection.** The origin/scale must match the world coordinates
  ([C29.2](02-trackmaps.md)); changing them misaligns every overlay from the map imagery.
- **Re-skinning tiles doesn't change projection.** New tile art in the same grid keeps the transform valid
  ([C29.1](01-tiles.md)).
- **A resized/re-gridded map needs a new projection** — if you change the tile layout, the world→map transform
  must be recomputed to match.
- **Verify overlays align** — after any map edit, check the player marker sits where the car actually is.

---

### Key takeaways

- The map is the **horizontal (X-Y) projection** of the Z-up world — Rockport from above.
- World→map is a **scale/offset** affine transform (drop Z, shift by origin, scale to pixels).
- **Zoom** is a scale change (with matching tiles); **pan** is an offset change; the player stays centered via
  offset.
- Accuracy depends on the **shared world coordinate frame** — one projection places every entity consistently.
- Preserve the projection on edits; re-skins keep it; re-gridding the map requires recomputing the transform.

**Continue:** [C29.4 — Overlays: player, objectives, cops](04-overlays.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md)
