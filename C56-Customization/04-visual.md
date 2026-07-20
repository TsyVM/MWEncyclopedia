# C56.4 — Visual Customization

> **The one-sentence version:** visual customization makes the car yours — `Paint` (body colour), `Vinyls`/`Decals`
> (graphics), `Rims` (wheels), `Spoilers`/body kits (body parts), racing numbers, even HUD colour — each editing
> the car's visual data or mesh selection for the renderer.

[← C56.3 — Tuning sliders](03-tuning-sliders.md) · [Chapter 56 hub](C56-Customization.md) ·
[Next: C56.5 — Reading customization in RE →](05-reading-customization.md)

---

## The visual layers

Visual customization ([C56.1](01-two-customizations.md)) is a stack of *cosmetic* layers, each a `Customize*`
category ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)):

| Layer | What it changes |
|---|---|
| `Paint` / `PaintDatum` | the body colour (and finish) |
| `Vinyls` | large graphic designs on the body |
| `Decals` | smaller stickers/logos |
| `CustomizeNumbers` | racing numbers |
| `Rims` | the wheels |
| `Spoilers` / `bodykit` | body parts (spoilers, kits, bumpers) — mesh changes |
| `CustomizeHUDColor` | even the HUD colour |

So a customised car is layered: a base *paint*, over which go *vinyls* and *decals*, with chosen *rims* and *body
parts* ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)). Each layer is independent — you mix paint,
graphics, wheels, and kit to build a distinctive look.

> ✅ *Verified:* the visual categories — `Paint`/`PaintDatum`, `Vinyls`, `Decals`, `CustomizeNumbers`, `Rims`,
> `Spoilers`, `bodykit`, `CustomizeHUDColor` — are present in `speed.exe` as `Customize*` screens/data.

## Paint, vinyls, decals: the material layer

The *material* layers — paint, vinyls, decals — customise the car's **surface appearance** without changing its
geometry:

- **Paint** ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) sets the body colour and finish (gloss,
  metallic) — a material parameter the paint effect ([C51.3](../C51-Render-Pipeline/03-effect-system.md)) uses.
- **Vinyls** are large graphic layers — applied over the paint as decal geometry/textures
  ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).
- **Decals** and **numbers** are smaller graphics — logos, sponsor stickers, racing numbers — layered on top.

These edit the car's *visual vault data* ([C56.1](01-two-customizations.md)) — which paint colour
([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)), which vinyl/decal textures
([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)), where placed. The renderer
([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) then draws the car with the chosen material and decal
layers. So the material layers are *data* edits (colour, texture selection) — no geometry change, just appearance.

## Rims, spoilers, kits: the geometry layer

The *geometry* layers — rims, spoilers, body kits — change the car's **actual shape**:

- **Rims** swap the wheel geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) — different wheel
  models.
- **Spoilers** / **body kits** swap or add body parts — a different front bumper, side skirts, a rear wing — each a
  mesh ([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)) selection.

Unlike the material layers ([above](#paint-vinyls-decals-the-material-layer)), these change the *mesh* — the car is
drawn with different geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) for the chosen parts. So a
body kit selects an alternate set of body meshes; the renderer draws them instead of the stock parts. This is *mesh
selection* driven by the customization data — the car's visual vault names which body/wheel meshes to use, and
customization edits that selection. Some kits have minor *aero* effects (a spoiler adding downforce) — a small
performance tie ([C56.1](01-two-customizations.md)) — but they're primarily cosmetic.

## Why deep visual customization

Most Wanted's deep visual customization ([above](#the-visual-layers)) serves the game's *identity* and *culture*:

- **Ownership and expression.** Building a distinctive car — your paint, your vinyls, your kit — makes it *yours*,
  investing you in it ([C54.3](../C54-GameFlow-Blacklist/03-the-blacklist.md)) as you climb.
- **The tuner-culture context.** MW05 sits in the mid-2000s import/tuner scene; deep visual customization (vinyls,
  kits, rims) *is* that culture, and the game leans into it as a core feature, not an afterthought.
- **Layered, data-driven.** Because each layer is *data* ([C56.1](01-two-customizations.md)) — paint colour, decal
  placement, mesh selection — the customization is deep *and* cheap to implement (compose existing rendering,
  [Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md), from data). No special code per look.

So visual customization is where the game's *culture* meets its *architecture*: the tuner-scene identity expressed
through the data-driven rendering ([C56.1](01-two-customizations.md)). It's a defining feature of the mid-2000s NFS
games, and MW's implementation — layered material and geometry customization, all vault-driven — is a clean example
of building a deep cosmetic system on a data-driven renderer. Your car becomes a statement, built from data.

## RE implications

- **Visual layers** — paint, vinyls, decals, numbers (material) + rims, spoilers, kits (geometry) + HUD colour.
- **Material layers edit appearance** — paint colour, decal textures — visual vault data, no geometry change.
- **Geometry layers edit the mesh** — rims/kits select alternate body/wheel meshes
  ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)).
- **Deep customization** expresses the tuner culture, built cheaply on the data-driven renderer.

---

### Key takeaways

- Visual customization is a **stack of cosmetic layers** — `Paint`, `Vinyls`, `Decals`, `CustomizeNumbers` (surface)
  + `Rims`, `Spoilers`, `bodykit` (geometry) + even `CustomizeHUDColor`.
- **Material layers** (paint/vinyls/decals) edit the car's **appearance** — colour and decal textures
  ([Chapters 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)–[7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) —
  no geometry change.
- **Geometry layers** (rims/spoilers/kits) edit the **mesh** — selecting alternate body/wheel geometry
  ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) the renderer draws.
- Each layer is **independent data** — mix paint, graphics, wheels, and kit — so customization is **deep yet
  data-driven** (no code per look).
- Deep visual customization expresses the **mid-2000s tuner culture** through the data-driven renderer — your car
  as a statement, built from vault data.

**Continue:** [C56.5 — Reading customization in RE](05-reading-customization.md) · [Chapter 56 hub](C56-Customization.md)
