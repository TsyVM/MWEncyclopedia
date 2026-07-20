# C51.4 — VisualTreatment: MW's Signature Look

> **The one-sentence version:** Most Wanted's unmistakable appearance — warm sepia/desaturated tint, heavy bloom,
> motion blur, vignette — is a `VisualTreatment` post-processing pass (via `VisualTreatmentShader`) applied over
> the whole rendered frame, and it changes with gameplay state (free-roam vs. pursuit).

[← C51.3 — The effect & shader system](03-effect-system.md) · [Chapter 51 hub](C51-Render-Pipeline.md) ·
[Next: C51.5 — The render frame →](05-render-frame.md)

---

## The look that defines the game

If you've seen Most Wanted, you remember its *look*: a warm, slightly desaturated, sepia-tinted world, blooming
highlights, motion blur at speed, and a darkened vignette at the edges. That look is not in the textures or the
models — it's a **post-processing pass** applied over the finished frame, called the **`VisualTreatment`** (driven
by **`VisualTreatmentShader`**). It's the single most recognisable rendering feature of the game.

- **Colour grading** — the sepia/teal tint and desaturation that give the world its mood.
- **Bloom** — bright areas bleed light, giving the sun-baked, glowing highlights.
- **Motion blur** — speed smears the image, selling velocity.
- **Vignette** — darkened corners that focus the eye and add grit.

> ✅ *Verified:* `VisualTreatment` and `VisualTreatmentShader` are present as strings in `speed.exe` — the named
> post-processing system. It runs as a final full-frame shader pass ([C51.5](05-render-frame.md)).

## How post-processing works

The visual treatment is a **full-screen pass** that runs *after* the scene is rendered
([C51.5](05-render-frame.md)):

```
1. render the 3D scene   → an off-screen render target (the raw frame)
2. VisualTreatment pass   → sample that frame, apply colour grade + bloom + blur + vignette
   → the graded frame     → the back buffer, presented to the screen
```

The scene is drawn into an off-screen texture, then the `VisualTreatmentShader` ([C51.3](03-effect-system.md))
processes *that whole image* — reading each pixel, applying the colour transform, adding the bloomed bright pass,
blending motion blur, and darkening the vignette — writing the result to the screen. So the treatment operates on
the *2D image*, not the 3D scene: it's a filter over the rendered frame, which is why it affects everything
uniformly (the whole world gets the tint). This is the standard post-processing model, and MW leans on it heavily
for its identity.

> 🟡 *Reasoned:* the render-to-texture-then-post-process pipeline is the standard post-processing model, consistent
> with the verified `VisualTreatment`/`VisualTreatmentShader` and the D3DX effect system; the exact treatment
> operations and their order are per-shader RE. The visual-treatment system's existence and role are verified.

## State-driven: the pursuit look

Crucially, the visual treatment is **state-driven** — it *changes* with gameplay, which is a large part of why MW
*feels* different in different situations:

- **Free-roam** — a warmer, brighter, calmer grade.
- **Pursuit** — as Heat rises ([Chapter 48](../C48-Pursuit-Heat/02-heat.md), `SetWorldHeat`), the treatment shifts
  — cooler, harsher, more contrast, heavier vignette — signalling danger *through the image itself*.
- **Speed/nitrous** — motion blur and effects intensify with velocity and NOS
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)).

So the visual treatment is not a fixed filter — it's a *dynamic* one, its parameters driven by game state (Heat,
speed, mode). This ties the *rendering* to the *gameplay*: the world literally *looks* more threatening in a
high-Heat pursuit. That the treatment responds to `SetWorldHeat` ([Chapter 48](../C48-Pursuit-Heat/02-heat.md))
means the pursuit system reaches into the renderer to set the mood — a cross-system link that makes the chase feel
visceral. The look isn't decoration; it's *feedback*.

## Why post-processing for the identity

Putting MW's identity in a post-processing pass (rather than baking it into assets) is a smart, flexible design:

- **Uniform and consistent.** Every surface, model, and texture gets the same treatment automatically — the world
  is visually coherent without art-directing each asset to match.
- **Dynamic and cheap to change.** The grade can shift with state ([above](#state-driven-the-pursuit-look))
  instantly, and the whole look can be retuned by editing the treatment parameters — no asset changes.
- **Separable.** The raw scene render is untouched; the treatment is a layer on top. Artists build assets in
  neutral colour, and the treatment stamps the mood — a clean division of labour.

So the `VisualTreatment` is the rendering-layer counterpart to the data-over-code pattern
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)): a fixed post-process shader, driven by
state/parameter data, that gives the game its consistent, dynamic identity over neutral assets. It's arguably the
most *characteristic* single system in the renderer — the thing that, more than any model or texture, makes a frame
unmistakably Most Wanted.

## RE implications

- **`VisualTreatment`** is a full-frame post-processing pass (`VisualTreatmentShader`) — MW's signature look
  (sepia grade, bloom, motion blur, vignette).
- **Post-processing** — render the scene to a texture, then filter the whole image before presenting.
- **State-driven** — the grade shifts with Heat ([Chapter 48](../C48-Pursuit-Heat/02-heat.md)), speed, and mode —
  the world *looks* more threatening in a pursuit.
- **Identity as a layer** — uniform, dynamic, separable — the rendering-side data-over-code pattern.

---

### Key takeaways

- Most Wanted's unmistakable look is a **`VisualTreatment`** post-processing pass (`VisualTreatmentShader`) — warm
  sepia grade, bloom, motion blur, vignette — applied over the finished frame.
- It's a **full-screen filter**: render the scene to a texture, then the treatment shader grades the whole 2D image
  before presenting — affecting everything uniformly.
- It's **state-driven** — the grade shifts with **Heat** (`SetWorldHeat`), speed, and mode, so the world *looks*
  more threatening in a high-Heat pursuit (the look is **feedback**, not decoration).
- Putting the identity in a **post-process layer** (over neutral assets) makes it uniform, dynamic, and retunable
  without touching art.
- It's the renderer's **data-over-code** identity system — arguably the most *characteristic* single feature of
  MW's rendering.

**Continue:** [C51.5 — The render frame](05-render-frame.md) · [Chapter 51 hub](C51-Render-Pipeline.md)
