# C51.6 — Reading the Renderer in RE

> **The one-sentence version:** navigate the renderer by the D3D9/D3DX imports (`Direct3DCreate9`, the Effect and
> math functions), the render classes (`RenderObject`/`RenderMethod`, the slot pools), the shader-model strings,
> and `VisualTreatment` — reading the render pipeline from API up to the signature look.

[← C51.5 — The render frame](05-render-frame.md) · [Chapter 51 hub](C51-Render-Pipeline.md) ·
[Next: Chapter 52 — Effects, Particles & the FX Bank →](../C52-Effects-Particles/C52-Effects-Particles.md)

---

## Anchors for render RE

The renderer is anchored on verified imports, classes, and strings:

- **The D3D9/D3DX imports** — `Direct3DCreate9`, `d3d9.dll`, `d3dx9_26.dll`,
  `D3DXCreateEffectFromResourceA`/`Pool`, the D3DX math ([C51.1](01-d3d9-foundation.md), [C51.3](03-effect-system.md)).
- **The render classes** — `RenderObject`, `RenderInfo`, `RenderMethod`, `RenderConn`/`eRenderConn`, `RenderingCar`
  ([C51.2](02-render-objects.md)).
- **The pools** — `RenderObjectSlotPool`, `RenderEPolySlotPool` (+Overflow) ([C51.2](02-render-objects.md)).
- **The shader models** — ps_1_1…ps_3_0, vs_1_1/vs_3_0 ([C51.1](01-d3d9-foundation.md)).
- **The look** — `VisualTreatment`, `VisualTreatmentShader` ([C51.4](04-visual-treatment.md)).

From these, the renderer is navigable: the API, the object model, the shading, and the signature pass.

## The RE workflow

Reading the renderer:

1. **Find the D3D9 device** — trace `Direct3DCreate9` to the device creation and the render-state setup
   ([C51.1](01-d3d9-foundation.md)).
2. **Map the render objects** — the `RenderObject`/`RenderMethod` classes and their pools
   ([C51.2](02-render-objects.md)); how sim state becomes drawable via `RenderConn`.
3. **Trace the effect system** — `D3DXCreateEffectFromResourceA` to the loaded effects; how materials select
   techniques ([C51.3](03-effect-system.md)).
4. **Follow the frame** — the view, matrices, cull, sort, submit, and the `VisualTreatment`
   ([C51.5](05-render-frame.md)).

The output is the full render picture: device, objects, shading, frame, and look.

## The imports are the map

A helpful RE property: the **imported D3D9/D3DX function names are a map** of what the renderer does
([C51.1](01-d3d9-foundation.md)). Unlike internal engine functions (which are anonymous addresses), the imports are
*named by Microsoft* — so seeing `D3DXCreateEffectFromResourceA` in the import table *tells you* the game uses the
Effect framework; `D3DXMatrixPerspectiveLH` tells you it builds left-handed projections; `Direct3DCreate9` tells
you it's D3D9. The import table is a labelled inventory of the renderer's API usage — a free, high-level map of its
capabilities. Reading it first ([C51.1](01-d3d9-foundation.md)) orients you before diving into the anonymous engine
code. This is the render-side counterpart to the self-documenting cop strings
([C49.6](../C49-Cops-Dispatch-Roadblocks/06-reading-fleet.md)): the API names narrate the system.

## The renderer consumes the whole book

The renderer is where the book's asset chapters converge — it *draws* everything the earlier chapters decoded:

- **Geometry** ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) and **meshes**
  ([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)) — the vertex/index data the draws submit.
- **Textures** ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) and **materials**
  ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) — bound to the effects.
- **The world** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) and **scenery**
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) — culled and drawn.
- **The simulated cars** ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) — fed through
  `RenderConn` ([C51.2](02-render-objects.md)).

So the renderer is the *consumer* of the whole asset and simulation pipeline — everything the game loads and
computes exists to reach this layer and become pixels. Reading the renderer ties the book together: it's where the
formats ([Part I–II](../C6-Texture-Codecs/C6-Texture-Codecs.md)), the world ([Part III](../C15-Track-Streaming/C15-Track-Streaming.md)),
and the simulation ([Part VIII](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) all come together on screen.

## RE implications

- **Anchor on** the D3D9/D3DX imports, the render classes and pools, the shader models, and `VisualTreatment`.
- **The RE workflow** — device → render objects → effects → the frame.
- **The imports are the map** — Microsoft-named functions narrate the renderer's API usage.
- **The renderer consumes the whole book** — geometry, textures, world, and cars converge here.

---

### Key takeaways

- The renderer is anchored on the **D3D9/D3DX imports**, the **render classes** (`RenderObject`/`RenderMethod`) and
  **pools**, the **shader models**, and **`VisualTreatment`**.
- The RE workflow: **device (`Direct3DCreate9`) → render objects → effect system → the render frame**.
- The **imported D3D9/D3DX names are a free high-level map** — `D3DXCreateEffectFromResourceA` proves the Effect
  framework, `…PerspectiveLH` proves left-handed projection — the API narrates the renderer.
- The renderer is the **consumer of the whole book** — geometry, meshes, textures, materials, world, scenery, and
  simulated cars all converge here to become pixels.
- Reading it **ties the book together** — where the formats, the world, and the simulation meet on screen.

**Next:** [Chapter 52 — Effects, Particles & the FX Bank](../C52-Effects-Particles/C52-Effects-Particles.md): the
smoke, sparks, and debris layered over the rendered scene.

**Sources:** `speed.exe` (verified: `Direct3DCreate9`, `d3d9.dll`, `d3dx9_26.dll`, `D3DXCreateEffectFromResourceA`/
`D3DXCreateEffectPool`, D3DX math incl. `D3DXMatrixPerspectiveLH`; shader models `ps_1_1`/`ps_1_4`/`ps_2_0`/`ps_2_b`/
`ps_3_0`/`vs_1_1`/`vs_3_0`; render classes `RenderObject`/`RenderInfo`/`RenderMethod`/`RenderConn`/`eRenderConn`/
`RenderingCar`; pools `RenderObjectSlotPool`/`RenderEPolySlotPool`; view `eViewTrack`; `VisualTreatment`/
`VisualTreatmentShader`).
