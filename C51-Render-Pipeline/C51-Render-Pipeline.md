# Chapter 51 — The Render Pipeline

> **Goal of this chapter:** decode how Most Wanted turns the world into pixels — the Direct3D 9 foundation
> (`d3d9.dll`, `d3dx9_26`, shader models ps_1_1→ps_3_0), the pooled render objects (`RenderObject`/`RenderMethod`,
> the slot pools), the D3DX Effect-framework materials, and MW's signature `VisualTreatment` post-processing that
> gives the game its unmistakable look.

Everything the earlier chapters built — geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)),
textures ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)), the simulated cars
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), the streamed world
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — exists to be *drawn*. This chapter decodes the
renderer: the Direct3D 9 device it runs on, the render-object model it draws, the shader/effect system that
materials it, and the `VisualTreatment` pass that stamps MW's sepia-tinted, bloomed, motion-blurred signature over
the whole frame. It's the layer that makes the game *look* like Most Wanted.

> **Verified against the executable.** Most Wanted renders on **Direct3D 9**: `Direct3DCreate9`, `d3d9.dll`, and
> `d3dx9_26.dll` (D3DX9 build 26) are imported; the D3DX9 Shader Compiler (9.04.91) built its shaders. It supports
> **shader models ps_1_1, ps_1_4, ps_2_0, ps_2_b, ps_3_0** and **vs_1_1, vs_3_0** — the full 2005 range. It uses
> the **D3DX Effect framework** (`D3DXCreateEffectFromResourceA`, `D3DXCreateEffectPool`) and D3DX math
> (`D3DXMatrixPerspectiveLH`, `D3DXVec3Transform`, …). Render objects are **pooled**: `RenderObjectSlotPool`,
> `RenderEPolySlotPool` (+Overflow). The render classes `RenderInfo`, `RenderMethod`, `RenderConn`/`eRenderConn`,
> `RenderingCar`, and MW's `VisualTreatment`/`VisualTreatmentShader` are present.

---

## Deep-dive pages

- [C51.1 — The Direct3D 9 foundation](01-d3d9-foundation.md): the device, DLLs, and shader models.
- [C51.2 — Render objects & pools](02-render-objects.md): `RenderObject`/`RenderMethod` and the slot pools.
- [C51.3 — The effect & shader system](03-effect-system.md): D3DX Effects, materials to shaders.
- [C51.4 — VisualTreatment: MW's signature look](04-visual-treatment.md): the post-processing that defines the game.
- [C51.5 — The render frame](05-render-frame.md): the view, matrices, and draw submission each frame.
- [C51.6 — Reading the renderer in RE](06-reading-render.md): navigating the render system.

---

## 51.1 The Direct3D 9 foundation

Most Wanted (2005 PC) renders on **Direct3D 9** ([C51.1](01-d3d9-foundation.md)) — `Direct3DCreate9` creates the
interface, `d3d9.dll` and `d3dx9_26.dll` are the runtime, and the game targets **shader models ps_1_1 through
ps_3_0** (and vs_1_1/vs_3_0), covering the wide range of 2005 GPUs from fixed-ish SM1 cards to SM3-capable
hardware. This is the graphics API the whole renderer sits on — a fixed foundation of the era.

## 51.2 Render objects & pools

The renderer draws **render objects** ([C51.2](02-render-objects.md)) — `RenderObject` instances carrying a
`RenderInfo` (what to draw) and a `RenderMethod` (how to draw it). Crucially, they're **pooled**:
`RenderObjectSlotPool` and `RenderEPolySlotPool` (with `…Overflow`) are pre-sized allocators
([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — the renderer allocates draw objects from fixed
pools, no per-frame heap churn. The `RenderConn`/`eRenderConn` connector ([C39.5](../C39-Vehicle-Simulation/05-connectors.md))
is how a simulated object (a car, `RenderingCar`) hands its state to the renderer.

## 51.3 The effect & shader system

Materials become pixels through the **D3DX Effect framework** ([C51.3](03-effect-system.md)):
`D3DXCreateEffectFromResourceA` loads compiled `.fx` effects (techniques and passes), and `D3DXCreateEffectPool`
shares parameters across them. A material ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) selects an
effect and binds its textures ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) and parameters; the effect's
vertex/pixel shaders (compiled to the supported models, [C51.1](01-d3d9-foundation.md)) run on the GPU to shade
each surface. `ShaderDepth` and the `RenderMethod` select which shader path a draw uses.

## 51.4 VisualTreatment: the signature look

MW's unmistakable appearance — the warm sepia/desaturated tint, heavy bloom, motion blur, and vignette — is a
**`VisualTreatment`** post-processing pass ([C51.4](04-visual-treatment.md)), driven by `VisualTreatmentShader`.
After the scene is rendered, the visual treatment reprocesses the whole frame (colour grading, bloom, blur) to
stamp the game's mood over it. The treatment *changes* with gameplay — the world looks different in a pursuit
([SetWorldHeat](../C48-Pursuit-Heat/02-heat.md)) than in free-roam — because the visual treatment is state-driven.
This pass is, more than any asset, what makes a screenshot *read* as Most Wanted.

## 51.5 The render frame

Each frame ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)), the renderer sets up a **view**
(`eViewTrack`), builds the camera matrices with D3DX (`D3DXMatrixPerspectiveLH` — note left-handed projection,
[C51.5](05-render-frame.md)), culls and gathers the visible render objects
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)), submits their draws (each binding its effect and shader),
and finally applies the `VisualTreatment` ([C51.4](04-visual-treatment.md)) before presenting. This is the render
phase of `FrameTick` — the per-frame journey from world state to the pixels on screen.

---

### Key takeaways

- Most Wanted renders on **Direct3D 9** (`d3d9.dll`, `d3dx9_26`, `Direct3DCreate9`), targeting **shader models
  ps_1_1→ps_3_0 / vs_1_1→vs_3_0** — the full 2005 range.
- It draws **pooled render objects** (`RenderObject`/`RenderInfo`/`RenderMethod`, `RenderObjectSlotPool`/
  `RenderEPolySlotPool`) fed by the `RenderConn` connector — no per-frame heap churn.
- Materials become pixels via the **D3DX Effect framework** (`.fx` techniques/passes,
  `D3DXCreateEffectFromResourceA`) with vertex/pixel shaders.
- **`VisualTreatment`** — the state-driven post-processing (sepia tint, bloom, motion blur, vignette) — is MW's
  **signature look**, stamped over every frame.
- The **render frame** sets a view, builds D3DX (left-handed) matrices, culls, submits draws, and applies the
  visual treatment before presenting.

**Next:** [Chapter 52 — Effects, Particles & the FX Bank](../C52-Effects-Particles/C52-Effects-Particles.md): the
smoke, sparks, and debris.
