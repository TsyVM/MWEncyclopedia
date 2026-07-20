# C29.4 — Overlays: Player, Objectives, Cops

> **The one-sentence version:** over the static map tiles the game draws live icons — the player, objectives,
> and cops — each a world entity projected to map space and drawn as a marker, updated every frame.

[← C29.3 — Positioning: world → map](03-world-to-map.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md) ·
[Next: C29.5 — The road network on the map →](05-road-overlay.md)

---

## The dynamic layer

The map has two layers: the **static tiles** ([C29.1](01-tiles.md)) — the map picture — and the **dynamic
overlays** — the live icons drawn on top. The overlays are what make the map *useful in the moment*: they show
where things are *now*, not just what the city looks like. Each overlay icon is a world entity projected to map
space ([C29.3](03-world-to-map.md)) and drawn as a marker from the HUD/UI atlases
([C27.2](../C27-FrontEnd-Shell-UI/02-ui-atlases.md)).

## The icons

The overlays a player relies on:

- **The player marker** — at the player's projected position, usually an arrow oriented to the car's heading, so
  you see both where you are and which way you face. Often the map is rotated or centered on this.
- **Objectives** — race start/finish flags, event markers, and points of interest
  ([plan Chapter 55](../README.md)) — where to go.
- **Cops** — during a pursuit ([Chapter 14](../C14-Vault-Pursuit-Surfaces/01-pursuit-ai.md)), the police
  vehicles as icons, so you can see the chase around you (subject to whether the game reveals them at the
  current heat).
- **The GPS destination** — the target of the route ([C29.5](05-road-overlay.md)).

Each is the same construction: take the entity's world position, project it ([C29.3](03-world-to-map.md)), draw
its icon at that map pixel.

## Projected, every frame

Overlays are **live** — recomputed each frame as entities move ([C27.4](../C27-FrontEnd-Shell-UI/04-hud.md)):

```python
def draw_overlays(map_view, world):
    for entity in world.tracked_entities():       # player, cops, objectives
        mx, my = world_to_map(entity.pos, map_view)   # project (C29.3)
        draw_icon(entity.icon, mx, my, entity.heading)
```

The player moves → its marker moves; a cop closes in → its icon closes in on the map; you approach an objective
→ its flag nears the player marker. This continuous projection is what makes the map a real-time picture of the
world around you.

> 🟡 *Reasoned:* the overlay set (player/objectives/cops/GPS) and per-frame projection are the standard
> game-map dynamic layer, consistent with the verified projection ([C29.3](03-world-to-map.md)), HUD atlases
> ([C27.4](../C27-FrontEnd-Shell-UI/04-hud.md)), and pursuit system
> ([Chapter 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)); the icons and projection are
> grounded, the exact overlay data is the map/HUD detail.

## Cops and heat

The cop overlay ties to the pursuit/heat system ([C14.2](../C14-Vault-Pursuit-Surfaces/02-heat-bounty.md)):
whether and how cops appear on the map can depend on heat and pursuit state — at higher heat the game may show
more of the pursuit, or the perception model may hide/reveal cops. So the cop overlay is not just "draw all
cops" but a gameplay-tuned reveal, driven by the pursuit vault ([Chapter 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)).
This connects the map to the pursuit gameplay: the map shows the chase as the pursuit rules dictate.

## Overlays are HUD elements

The overlays are HUD/UI elements ([C27.4](../C27-FrontEnd-Shell-UI/04-hud.md)) — icons from atlases positioned
by the world→map projection instead of a static layout. So editing them is the HUD-editing path
([C27.6](../C27-FrontEnd-Shell-UI/06-editing-ui.md)): the icon art is atlas TPK, and the positioning is the
projection ([C29.3](03-world-to-map.md)). Re-skinning the player arrow, objective flags, or cop icons is an
atlas edit.

## Editing implications

- **Re-skin icons via the atlas** — the player marker, objective flags, cop icons are atlas art
  ([C27.2](../C27-FrontEnd-Shell-UI/02-ui-atlases.md)).
- **Positioning is the projection** — icons follow world→map ([C29.3](03-world-to-map.md)); don't hand-place
  them.
- **Cop reveal is gameplay-tuned** — how cops show is tied to pursuit/heat
  ([Chapter 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)), not just the map.
- **Verify alignment** — icons must sit at their entities' real positions ([C29.3](03-world-to-map.md)).

---

### Key takeaways

- The map has a **static** tile layer and a **dynamic** overlay layer of live icons.
- Overlays — player, objectives, cops, GPS destination — are world entities projected to map space each frame.
- Continuous projection makes the map a real-time picture: things move on the map as they move in the world.
- The **cop** overlay is tied to pursuit/heat (a gameplay-tuned reveal), linking the map to the chase.
- Overlays are HUD elements: re-skin icons via the atlas; positioning is the world→map projection.

**Continue:** [C29.5 — The road network on the map](05-road-overlay.md) · [Chapter 29 hub](C29-Minimap-Map-Data.md)
