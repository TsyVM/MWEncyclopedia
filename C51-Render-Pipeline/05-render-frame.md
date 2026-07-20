# C51.5 — The Render Frame

> **The one-sentence version:** each frame the renderer sets up a view (`eViewTrack`), builds the camera matrices
> with D3DX (left-handed projection), culls and gathers the visible render objects, submits their draws (each
> binding its effect and shader), and applies the `VisualTreatment` before presenting.

[← C51.4 — VisualTreatment](04-visual-treatment.md) · [Chapter 51 hub](C51-Render-Pipeline.md) ·
[Next: C51.6 — Reading the renderer in RE →](06-reading-render.md)

---

## The render phase of the frame

Rendering is a phase of `FrameTick` ([Chapter 37](../C37-Frame-Spine-Modules/04-frametick.md)) — after the
simulation has updated the world ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) and published
render state ([C51.2](02-render-objects.md)), the render phase turns that state into pixels. Its per-frame sequence:

```
render frame:
   1. set up the view       — eViewTrack: camera position/orientation (from CameraAI, Ch 53)
   2. build the matrices     — D3DXMatrixPerspectiveLH etc. (view + projection)
   3. cull & gather          — the visible render objects (cull tree, Ch 16)
   4. sort & batch           — by material/state to minimise GPU state changes
   5. submit draws           — each RenderObject binds its effect+shader, DrawPrimitive
   6. VisualTreatment        — the post-process pass (C51.4)
   7. present                — the finished frame to the screen
```

This is the journey from world state to screen, once per frame. Each step is a stage the renderer runs on the D3D9
device ([C51.1](01-d3d9-foundation.md)).

## The view and the camera

The frame begins with a **view** — `eViewTrack` (verified) is the gameplay view, defining *where the camera is* and
*what it sees*. The camera comes from `CameraAI` ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)) —
the automatic gameplay camera that follows your car. From the camera's position and orientation, the renderer
builds the **view matrix** (world → camera space) and the **projection matrix**
(`D3DXMatrixPerspectiveLH`, [C51.3](03-effect-system.md)) (camera → clip space). Together these transform world
geometry into the 2D frame. That there's a named *view* object means the renderer can support multiple views (the
main view, mirror/reflection views, the map) — each an `eView*` with its own camera and matrices.

> ✅ *Verified:* `eViewTrack` is a named view class; the projection uses `D3DXMatrixPerspectiveLH`
> ([C51.3](03-effect-system.md)) — the render frame is built around a view with D3DX-computed matrices. `RenderInfo`
> and `RenderMethod` ([C51.2](02-render-objects.md)) carry the per-object draw data.

## Cull, sort, submit

The heart of the frame is **cull → sort → submit**:

- **Cull** — determine which render objects are visible from the view, using the scenery cull tree
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) and section PVS ([C15.5](../C15-Track-Streaming/C15-Track-Streaming.md)).
  Only visible objects proceed — the whole city isn't drawn, just what the camera can see.
- **Sort/batch** — order the visible draws to minimise expensive GPU state changes: group by effect/shader
  ([C51.3](03-effect-system.md)) and render state, so the device switches shaders and textures as little as
  possible. This is where the render-object layer ([C51.2](02-render-objects.md)) pays off — with all objects
  gathered, they can be reordered for efficiency.
- **Submit** — for each render object, bind its effect+technique ([C51.3](03-effect-system.md)), set its material
  parameters and textures, bind its geometry ([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)), and issue the draw.

So the frame draws only what's visible, in an order that's cheap for the GPU. This cull-sort-submit core is common
to all real-time renderers, and MW's render-object model ([C51.2](02-render-objects.md)) exists to enable it.

> 🟡 *Reasoned:* the cull→sort→submit structure is the standard real-time render loop, consistent with the verified
> render-object model, the cull tree ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)), and the effect
> system; the exact sort keys and batching are deeper RE. The view, matrices, render classes, and visual treatment
> are verified.

## Post-process and present

After the scene draws, two final steps ([C51.4](04-visual-treatment.md)):

- **VisualTreatment** — the post-process pass ([C51.4](04-visual-treatment.md)) grades the whole rendered image
  (colour, bloom, blur, vignette) — MW's look, applied last.
- **Present** — the finished, graded frame is presented to the screen (the D3D9 `Present` call), and the next
  frame begins.

So the frame ends where the player sees it: a culled, sorted, drawn, and *treated* image, swapped to the display.
The whole render frame — view, matrices, cull, sort, submit, treat, present — runs every `FrameTick`
([Chapter 37](../C37-Frame-Spine-Modules/04-frametick.md)), turning the simulation's world state into the image on
screen. Combined with the sim ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) that produces
that state, this closes the loop from *input* to *pixels*.

## RE implications

- **The render frame** — view → matrices → cull → sort → submit → VisualTreatment → present — is a phase of
  `FrameTick`.
- **The view** (`eViewTrack`) + D3DX matrices (left-handed projection) define what's drawn.
- **Cull → sort → submit** — draw only the visible, ordered to minimise GPU state changes (why the render-object
  layer exists).
- **VisualTreatment then present** — the look applied last, the frame swapped to screen.

---

### Key takeaways

- Rendering is a **phase of `FrameTick`** — **view → matrices → cull → sort → submit → VisualTreatment → present**
  — turning published world state into pixels each frame.
- The frame begins with a **view** (`eViewTrack`) from `CameraAI` and **D3DX matrices**
  (`D3DXMatrixPerspectiveLH`, left-handed clip space).
- The core is **cull → sort → submit** — draw only the visible objects (cull tree,
  [Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)), sorted by effect/state to **minimise GPU state changes**
  (the payoff of the render-object layer).
- Each draw **binds its effect+shader and material**, then issues a D3D9 primitive.
- The frame ends with the **`VisualTreatment`** post-process ([C51.4](04-visual-treatment.md)) and **present** —
  closing the loop from world state to the image on screen.

**Continue:** [C51.6 — Reading the renderer in RE](06-reading-render.md) · [Chapter 51 hub](C51-Render-Pipeline.md)
