# C51.3 — The Effect & Shader System

> **The one-sentence version:** materials become pixels through the D3DX Effect framework —
> `D3DXCreateEffectFromResourceA` loads compiled `.fx` effects (techniques and passes), `D3DXCreateEffectPool`
> shares their parameters, and the effect's vertex/pixel shaders (ps_1_1→ps_3_0) shade each surface.

[← C51.2 — Render objects & pools](02-render-objects.md) · [Chapter 51 hub](C51-Render-Pipeline.md) ·
[Next: C51.4 — VisualTreatment →](04-visual-treatment.md)

---

## The D3DX Effect framework

Most Wanted shades surfaces with the **D3DX Effect framework** — the D3D9-era system for packaging shaders and
render state into reusable `.fx` "effects." The verified imports:

- **`D3DXCreateEffectFromResourceA`** — loads a compiled effect (a `.fx` with one or more *techniques*, each a set
  of *passes*, each pass a vertex + pixel shader and render state).
- **`D3DXCreateEffectPool`** — creates a shared parameter pool, so common parameters (the view/projection matrices,
  lighting) are set once and shared across all effects.

So the renderer's shading is organised as a library of effects. A `RenderMethod` ([C51.2](02-render-objects.md))
selects an effect and a technique within it; the technique's passes run the shaders that produce the pixels. This
is the standard D3D9 way to manage shaders, and MW uses it wholesale.

> ✅ *Verified:* `D3DXCreateEffectFromResourceA` and `D3DXCreateEffectPool` are imported by `speed.exe`, alongside
> the D3DX math functions — the renderer uses the D3DX Effect framework. `Shader`, `ShaderDepth`, and
> `VisualTreatmentShader` are named effects/shaders ([C51.4](04-visual-treatment.md)).

## Material → effect → shader

The path from a car's paint or a road's asphalt to shaded pixels is a chain:

```
material (Chapter 7)         — names an effect + textures + parameters
   → effect (.fx)             — a technique with vertex/pixel shader passes
      → shader (ps_/vs_)       — runs per-vertex and per-pixel on the GPU
         → shaded pixel        — the final colour for that surface point
```

A **material** ([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) is the data — it names which effect to
use, binds its textures ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)), and sets its parameters (colour,
shininess). The **effect** is the shading logic — the shader code and passes. So the material is the *what* (this
surface's textures and constants) and the effect is the *how* (the shading algorithm). Many materials share one
effect (all car paint uses the paint effect, with different colours); one material uses one effect. This
material-over-effect split is the same data-over-code pattern as the rest of the engine
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) — the effect is fixed shading code, the material
is per-surface data.

## Shader tiers and fallback

Because the game targets ps_1_1 through ps_3_0 ([C51.1](01-d3d9-foundation.md)), effects have **multiple
techniques** — one per shader tier — and the renderer selects the best the GPU supports:

- **High tier** (`ps_3_0`/`vs_3_0`) — the full effect: per-pixel lighting, complex materials, the works.
- **Mid tier** (`ps_2_0`) — a reduced version fitting the shorter shader limits.
- **Low tier** (`ps_1_1`) — a simplified approximation for older cards.

The D3DX Effect framework supports exactly this — an effect declares several techniques, and the app picks a valid
one at load. So a single "car paint" effect contains a ps_3_0 technique *and* a ps_1_1 fallback, and each GPU runs
the best it can. This is how MW rendered acceptably across the 2005 hardware spread
([C51.1](01-d3d9-foundation.md)) without shipping separate builds — the fallback is in the effect.

> 🟡 *Reasoned:* the multi-technique-per-effect fallback is the standard D3DX Effect approach to shader tiers,
> consistent with the verified shader-model range and the Effect framework usage; the exact technique set per
> effect is per-`.fx` RE. The Effect-framework imports and shader models are verified.

## D3DX math

The verified D3DX math functions (`D3DXMatrixPerspectiveLH`, `D3DXMatrixMultiply`, `D3DXVec3Transform`,
`D3DXVec3Normalize`, …) are the renderer's **linear algebra** — building the camera and object matrices that the
shaders use ([C51.5](05-render-frame.md)):

- **`D3DXMatrixPerspectiveLH`** / **`D3DXMatrixOrthoLH`** — the projection matrices (**left-handed** — a
  render-side convention, distinct from the game world's Z-up
  [Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).
- **`D3DXMatrixMultiply`/`Transpose`/`Inverse`** — composing and inverting transforms.
- **`D3DXVec3Transform`/`TransformCoordArray`** — transforming points (e.g. skinning, instancing).

That the game uses D3DX for this (rather than its own math) ties the render math to the D3DX library and its
conventions (left-handed matrices). The `LH` suffix is a notable detail: the *rendering* uses left-handed
projection even though the *simulation* world is Z-up — the render layer converts world coordinates into its own
clip space via these matrices. This is a normal engine seam (world space vs. clip space), verified here by the
`…LH` function names.

## RE implications

- **The D3DX Effect framework** shades surfaces — `.fx` effects (techniques/passes) via
  `D3DXCreateEffectFromResourceA`, shared params via `D3DXCreateEffectPool`.
- **Material → effect → shader** — material is per-surface data, effect is fixed shading code (data-over-code).
- **Multi-technique fallback** — each effect has tiers (ps_3_0 down to ps_1_1); the renderer picks the best the GPU
  supports.
- **D3DX math** builds the matrices — **left-handed** projection (render clip space), distinct from the Z-up world.

---

### Key takeaways

- Surfaces are shaded via the **D3DX Effect framework** — compiled `.fx` effects (techniques + passes) loaded with
  `D3DXCreateEffectFromResourceA`, sharing parameters through `D3DXCreateEffectPool`.
- The path is **material → effect → shader** — the **material** is per-surface data (textures, constants), the
  **effect** is fixed shading code (the data-over-code pattern again).
- Effects carry **multiple techniques** (ps_3_0 down to ps_1_1) and the renderer picks the best the GPU supports —
  the **fallback is in the effect**, so one build runs on all 2005 hardware.
- **D3DX math** builds the render matrices — **left-handed** projection (`…LH`), the render clip space distinct
  from the game's Z-up world.
- This is standard, verified **D3D9/D3DX** shading — a library of effects selected per material and GPU tier.

**Continue:** [C51.4 — VisualTreatment: MW's signature look](04-visual-treatment.md) · [Chapter 51 hub](C51-Render-Pipeline.md)
