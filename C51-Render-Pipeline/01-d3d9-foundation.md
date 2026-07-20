# C51.1 — The Direct3D 9 Foundation

> **The one-sentence version:** Most Wanted (2005 PC) renders on Direct3D 9 — `Direct3DCreate9` on `d3d9.dll` with
> `d3dx9_26.dll` — targeting shader models ps_1_1 through ps_3_0 (and vs_1_1/vs_3_0), the full range of 2005 GPU
> capability.

[← Chapter 51 hub](C51-Render-Pipeline.md) · [Next: C51.2 — Render objects & pools →](02-render-objects.md)

---

## The API

Most Wanted's renderer sits on **Direct3D 9**, the standard Windows graphics API of its era. The verified imports
tell the story:

- **`Direct3DCreate9`** — the entry point that creates the `IDirect3D9` interface, from which the game creates its
  rendering device.
- **`d3d9.dll`** — the Direct3D 9 runtime.
- **`d3dx9_26.dll`** — the D3DX9 utility library, **build 26** (a specific SDK release from 2005), providing math,
  texture, and effect helpers ([C51.3](03-effect-system.md)).

So the whole renderer is a Direct3D 9 application: it creates a D3D9 device, sets render state, binds shaders and
textures, and submits primitives — the classic immediate-mode D3D9 model. This foundation is fixed (the game was
built for D3D9), and everything above it — render objects, effects, the visual treatment — is built on the D3D9
device.

> ✅ *Verified:* `Direct3DCreate9`, `d3d9.dll`, and `d3dx9_26.dll` are imported by `speed.exe`; the D3DX9 Shader
> Compiler (version 9.04.91) is referenced — the game is a Direct3D 9 / D3DX9 build-26 application.

## Shader models: ps_1_1 to ps_3_0

The verified shader-model strings reveal the renderer's **shader range** — it compiles and supports multiple
profiles to cover the wide spread of 2005 hardware:

| Profile | Capability | Era hardware |
|---|---|---|
| `ps_1_1`, `ps_1_4` | pixel shader 1.x — limited instructions | GeForce 3/4, Radeon 8500 |
| `ps_2_0`, `ps_2_b` | pixel shader 2.x — floating-point, longer | GeForce FX, Radeon 9000 |
| `ps_3_0` | pixel shader 3.0 — dynamic branching, long | GeForce 6/7, Radeon X1000 |
| `vs_1_1`, `vs_3_0` | vertex shader 1.1 / 3.0 | matching tiers |

Supporting `ps_1_1` *through* `ps_3_0` means the renderer has **multiple shader paths** — a high-end path
(`ps_3_0`/`vs_3_0`) with full effects for capable cards, and fallback paths (`ps_1_1`) for older hardware that
render a simpler approximation. This is how a 2005 game ran on both a top GeForce 7 and a three-year-old GeForce 4:
the effect system ([C51.3](03-effect-system.md)) selects the best shader path each card supports.

> ✅ *Verified:* the shader-model profiles `ps_1_1`, `ps_1_4`, `ps_2_0`, `ps_2_b`, `ps_3_0`, `vs_1_1`, `vs_3_0` are
> present as strings in `speed.exe` — the range of shader targets the renderer compiles/supports.

## The device and render state

On top of `Direct3DCreate9`, the game creates a **D3D9 device** — the object that owns the GPU connection and all
rendering. The device is configured with:

- **A back buffer and depth buffer** — the render targets the frame is drawn into ([C51.5](05-render-frame.md)).
- **Render state** — blend modes, depth testing, culling, etc., set per draw batch.
- **Bound resources** — the current vertex/index buffers ([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)),
  textures ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)), and shaders.

Every draw the renderer issues goes through this device: set the state, bind the buffers and shaders, call
`DrawPrimitive`. So the device is the renderer's handle to the GPU, and the whole render frame
([C51.5](05-render-frame.md)) is a sequence of device operations. The render-object model ([C51.2](02-render-objects.md))
and effect system ([C51.3](03-effect-system.md)) are abstractions *over* this device, organising what state to set
and what to draw.

> 🟡 *Reasoned:* the device configuration (back/depth buffers, render state, resource binding) is the standard D3D9
> application model, consistent with the verified `Direct3DCreate9` and the render classes; the exact device
> parameters and present flags are per-config RE. The API, DLLs, and shader models are verified.

## Why D3D9 shaped the engine

Building on Direct3D 9 (rather than an abstraction over multiple APIs) shaped the renderer in ways visible
throughout:

- **The Xbox/PC lineage.** MW05 shipped on Xbox, PS2, GameCube, and PC; the PC build's D3D9 path is one target of a
  multi-platform engine ([C41.6](../C41-Physics-RigidBody/06-vehicle-types.md) noted the same shared-engine
  breadth). The `eRenderConn`/`RenderMethod` abstraction ([C51.2](02-render-objects.md)) is partly to isolate the
  platform-specific graphics.
- **Shader-model tiering** ([above](#shader-models-ps_1_1-to-ps_3_0)) is a D3D9-era necessity — the wide GPU spread
  of 2005 demanded fallbacks.
- **The D3DX dependency** ([C51.3](03-effect-system.md)) — using D3DX for math and effects is a D3D9 convenience
  that ties the build to a specific D3DX version (`d3dx9_26`).

So the D3D9 foundation is not incidental — it's the technological context that explains the renderer's structure
(platform abstraction, shader tiers, D3DX effects). Understanding the renderer starts with understanding it's a
2005 D3D9 application, with all that implies.

## RE implications

- **The renderer is Direct3D 9** — `Direct3DCreate9`, `d3d9.dll`, `d3dx9_26.dll` (build 26).
- **Shader models ps_1_1→ps_3_0** — multiple paths for the wide 2005 GPU spread; the effect system picks the best.
- **A D3D9 device** owns the GPU; every draw sets state, binds resources, and issues a primitive.
- **D3D9 shaped the engine** — platform abstraction, shader tiering, the D3DX dependency.

---

### Key takeaways

- Most Wanted renders on **Direct3D 9** — `Direct3DCreate9`, `d3d9.dll`, and `d3dx9_26.dll` (**build 26**) —
  verified imports.
- It supports **shader models ps_1_1, ps_1_4, ps_2_0, ps_2_b, ps_3_0** and **vs_1_1, vs_3_0** — multiple paths so
  the game runs on both high-end and older 2005 GPUs.
- A **D3D9 device** owns the GPU connection; every draw sets render state, binds buffers/textures/shaders, and
  issues a primitive.
- The **D3D9 foundation shaped the engine** — the platform-abstracting render classes, the shader tiering, and the
  D3DX dependency all follow from it.
- Understanding the renderer starts with recognising it as a **2005 D3D9 application**.

**Continue:** [C51.2 — Render objects & pools](02-render-objects.md) · [Chapter 51 hub](C51-Render-Pipeline.md)
