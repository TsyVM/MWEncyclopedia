# C70.4 — Vinyls & Decals

> **The one-sentence version:** vinyls are *textures* — the per-car `VINYLS.BIN` is a texture pack of the vinyl
> artwork (276 textures for the M3 GTR), and `PREVINYL.BIN` holds the *pre-composited* result (3 textures) the game
> bakes your chosen, positioned vinyls into — so a livery is layered vinyl textures composited onto the body.

[← C70.3 — Paint & colour](03-paint.md) · [Chapter 70 hub](C70-Visual-Customisation.md) ·
[Next: C70.5 — Reading visual customisation in RE →](05-reading-visual.md)

---

## Vinyls are textures, not geometry

Where body kits are geometry ([C70.1](01-body-kits.md)) and paint is a material ([C70.3](03-paint.md)), vinyls and
decals are **textures** ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)). Each car ships a `VINYLS.BIN` — a
**texture pack** (`TPK`) of vinyl artwork. The retail `CARS/BMWM3GTR/VINYLS.BIN` is a `VINYL` pack of **276
textures** (internal path `…/BMWM3GTR_vinyls.tpk`): the shapes, logos, tribals, and patterns you can apply. Choosing a
vinyl selects one of these textures; positioning it (move, scale, rotate, colour via `VINYL_PAINT`
[C70.3](03-paint.md)) places it on the body.

> ✅ *Verified:* `VINYL` (×13) and `DECAL` (×94) are strings in `speed.exe`; retail `CARS/BMWM3GTR/VINYLS.BIN` is a
> `VINYL` `TPK` ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) of 276 textures, and `PREVINYL.BIN` a `VINYL`
> `TPK` of 3 (`…/BMWM3GTR_precompvinyls.tpk`).

## Pre-composition: the precomp vinyl

The key trick is **`PREVINYL.BIN`** — the *pre-composited* vinyls. A player's livery might layer dozens of vinyls,
each positioned and coloured; drawing all of them, every frame, on every car would be expensive. Instead the game
**bakes** the chosen, positioned, coloured vinyl textures into a small **pre-composited texture** (`PREVINYL.BIN`, 3
textures) — the finished livery flattened into one image the car wears. So there are two stages:

```
VINYLS.BIN (276 source vinyls)  ->  player selects & positions layers  ->  bake  ->  PREVINYL.BIN (composited livery)
```

`VINYLS.BIN` is the *palette* (all the artwork you *could* use); `PREVINYL.BIN` is the *painting* (the livery you
*did* make, flattened). This is a classic render-cost optimisation: composite once at customization time, then draw
the single precomp texture at race time — the same "bake the expensive thing ahead" logic as lightmaps or the
minimap ([Chapter 29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)). It's why a heavily-liveried car costs no
more to render than a plain one: both wear a single precomp texture.

> 🟡 *Reasoned:* the `VINYLS.BIN` = source palette / `PREVINYL.BIN` = baked-livery reading follows from the file
> names (`vinyls` vs `precompvinyls`), the texture counts (276 vs 3), and the render-cost logic; the exact
> composition pipeline is runtime/tool data. The two `TPK`s, their contents, and the `VINYL`/`DECAL` strings are
> verified.

## Decals and the mount meshes

`DECAL` (×94, the largest visual string family) is the decal machinery — and it has both a *texture* side and a
*geometry* side ([C70.1](01-body-kits.md)):

- **Texture** — the decal artwork lives in `VINYLS.BIN` alongside the vinyls (a decal is a kind of vinyl — a badge,
  a number, a logo).
- **Geometry** — the `KIT00_DECAL_*` meshes ([C70.1](01-body-kits.md)) are the *mount surfaces*: where on the body a
  decal can sit (`LEFT_DOOR`, `LEFT_QUARTER`, the windows). These are shaped to the kit so decals conform to the
  car's contours.

So placing a decal is texturing a mount mesh: the artwork (texture) is projected onto a decal-mount surface
(geometry) at your chosen position. Racing numbers (`CustomizeNumbers`,
[C68.2](../C68-Vehicles-Customisable-Object/02-shop-categories.md)) are the same mechanism with number glyphs. This
texture-on-mount-mesh design is why decals follow the body's shape and get occluded correctly — they're real surfaces
on the car, not screen-space stickers.

## The visual data trinity

Vinyls complete the three *kinds* of visual data a car carries:

- **Geometry** — body kits, wheels, aero ([C70.1](01-body-kits.md)–[C70.2](02-wheels-aero.md)): `GEOMETRY.BIN`.
- **Colour** — paint ([C70.3](03-paint.md)): material parameters.
- **Textures** — vinyls/decals (this page): `VINYLS.BIN` → `PREVINYL.BIN`, plus the base `TEXTURES.BIN`.

A finished custom car is the sum: a chosen body (geometry), painted (colour), wearing a baked livery (texture). The
three compose independently ([C70.3](03-paint.md)) — which is the whole power of MW's customization: mesh, colour,
and texture are separate layers you mix freely. Reading vinyls completes the visual picture: they're the *texture*
layer, sourced from `VINYLS.BIN` and baked to `PREVINYL.BIN`, laid over the painted geometry
([C70.5](05-reading-visual.md)).

## RE implications

- **Vinyls are textures** — `VINYLS.BIN` is a `TPK` ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) of vinyl
  artwork (276 for the M3 GTR); choosing/positioning one places it on the body.
- **Pre-composition** — `PREVINYL.BIN` is the **baked livery** (3 textures): composite the layered vinyls once, draw
  one texture at race time (a render-cost optimisation).
- **Decals = texture on mount mesh** — artwork (`VINYLS.BIN`) projected onto `KIT00_DECAL_*` surfaces
  ([C70.1](01-body-kits.md)); numbers are the same mechanism.
- **The visual data trinity** — geometry (`GEOMETRY.BIN`) + colour (paint) + textures (`VINYLS`/`PREVINYL`) — three
  independent layers.

---

### Key takeaways

- Vinyls and decals are **textures** ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) — the per-car `VINYLS.BIN`
  is a `TPK` **palette** of vinyl artwork (276 textures for the M3 GTR); choosing and positioning one places it on
  the body.
- **`PREVINYL.BIN` is the baked livery** — the game **pre-composites** your layered, positioned, coloured vinyls into
  a small texture (3 images), so a heavily-liveried car costs **no more to render** than a plain one.
- **`DECAL` (×94)** is texture *and* geometry — the artwork lives in `VINYLS.BIN`, projected onto **`KIT00_DECAL_*`
  mount meshes** ([C70.1](01-body-kits.md)) shaped to the kit, so decals conform and occlude like real surfaces.
- A car's visual data is a **trinity** — **geometry** (`GEOMETRY.BIN`) + **colour** (paint) + **textures**
  (`VINYLS`/`PREVINYL`) — three independent layers you mix freely.
- Verified: the `VINYL`/`DECAL` strings and the two vinyl `TPK`s (source `VINYLS.BIN` vs precomp `PREVINYL.BIN`) in
  retail car data.

**Continue:** [C70.5 — Reading visual customisation in RE](05-reading-visual.md) · [Chapter 70 hub](C70-Visual-Customisation.md)
