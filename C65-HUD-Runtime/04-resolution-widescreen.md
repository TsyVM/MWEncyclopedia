# C65.4 — Resolution & Widescreen

> **The one-sentence version:** normalized anchors make the HUD resolution-independent (fractions scale to any
> backbuffer), while aspect ratio is handled by *package selection* — `WIDESCREEN_GLOBAL` vs `THINSCREEN_GLOBAL`
> twin FE bundles (16:9 vs 4:3), the live choice visible as `FEngHud::mCurrentWidescreenSetting`.

[← C65.3 — Placement & anchoring](03-pursuit-hud.md) · [Chapter 65 hub](C65-HUD-Runtime.md) ·
[Next: C65.5 — Gauges & meters →](05-gauges-meters.md)

---

## Resolution: solved by normalization

Because placement is **normalized `0..1` screen space** ([C65.3](03-pursuit-hud.md)), *resolution* is handled
automatically. A widget's anchor is a *fraction* of the backbuffer, so:

```
pixel_position = normalized_anchor × current_backbuffer_size
```

The *same* layout data produces correct pixel positions at 640×480, 1024×768, 1280×1024, 1920×1080 — the fraction
is fixed, only the multiply changes. So there is **no per-resolution HUD data** — one set of normalized layouts
covers every display size the game supports. This is the payoff of normalized space
([C65.3](03-pursuit-hud.md)): resolution-independence *by construction*, no special cases. Change the resolution,
the HUD scales; nothing in the HUD data needs to know the resolution.

> ✅ *Verified:* the normalized-space model is the FEng idiom ([C65.3](03-pursuit-hud.md)); the aspect-twin bundles
> `WIDESCREEN_GLOBAL` and `THINSCREEN` are present in `speed.exe` (10 distinct `WS_*.fng` screens).

## Aspect ratio: package selection, not math

Resolution is one thing; **aspect ratio** (4:3 vs 16:9) is another — and it *can't* be solved by pure normalization,
because a normalized square `(0.2, 0.2)` is a *different shape* at 4:3 than at 16:9 (the pixels are non-square
relative to each other). MW05 solves this not with per-element aspect math but with **whole-package selection**:

- **`GLOBAL/WIDESCREEN_GLOBAL.BUN`** — the **16:9** FE bundle: FEPackages (`0x00030203`) laid out for widescreen.
- **`GLOBAL/THINSCREEN_GLOBAL.BUN`** — the **4:3** FE bundle: the *same screens* re-laid-out for standard.
- **`WS_*.fng`** — explicit widescreen twins of loading/boot screens (10 verified: `WS_Loading.fng`, `WS_LS_*.fng`,
  …).

So the game *loads a different layout package* depending on the display's aspect ratio — the 16:9 player gets the
widescreen layouts, the 4:3 player gets the thinscreen layouts, each hand-tuned for its aspect. The **live choice
is on the HUD object**: `FEngHud::mCurrentWidescreenSetting` ([C65.2](02-gauge-cluster.md)) records which is in use.

> 🟡 *Reasoned:* that the aspect twins are *whole re-laid-out packages* (not per-element scaling) is the natural
> reading of the verified `WIDESCREEN_GLOBAL`/`THINSCREEN_GLOBAL` bundle pair and the `WS_*.fng` twins; the exact
> selection logic is deeper RE. The bundle pair and `mCurrentWidescreenSetting` are verified/consistent.

## Why package selection for aspect

Handling aspect by *selecting a package* rather than *scaling elements* is a deliberate, pragmatic choice:

- **Hand-tuned layouts per aspect.** A widescreen HUD isn't just a stretched 4:3 HUD — the extra horizontal space
  is *used* (elements spread out, the minimap repositioned). A per-element scale couldn't do this; a separate
  layout can. So each aspect gets a *designed* layout, not a distorted one.
- **No runtime aspect math.** The renderer just applies the normalized layout ([C65.3](03-pursuit-hud.md)) it was
  given; it doesn't compute aspect corrections per element. The complexity is in *data* (two layouts), not *code*.
- **Clean authoring.** Artists lay out the 16:9 and 4:3 HUDs *separately*, each looking right, rather than fighting
  a single layout to work at both. This is the data-over-code pattern ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md))
  — the aspect variation is authored data.

So aspect is a *content* problem solved with *content* (two layout sets), not a *math* problem solved with *code*.
This is why the widescreen and standard HUDs both look *intentional* — each is a designed layout, selected by
display. It also defines a modding trap ([C65.8](08-reading-hud.md)): editing the *wrong* aspect's package is a
common reason a HUD change "doesn't show up" — you edited the 4:3 layout while playing 16:9, or vice versa.

## The resolution/aspect split

The two mechanisms compose cleanly ([above](#resolution-solved-by-normalization)):

```
display = (resolution, aspect_ratio)
   resolution → handled by normalized coords × backbuffer     (one layout, C65.3)
   aspect     → handled by selecting the WIDE or THIN package  (two layouts)
```

So **resolution** is a *rendering* concern (multiply by backbuffer — free, per-frame), and **aspect** is a *data*
concern (which package to load — chosen once). Together they cover the full space of displays: any resolution × 4:3
or 16:9. This split — normalize for size, select for aspect — is an elegant, minimal solution: one layout per
aspect (two total), scaled to any resolution. It's why the MW05 HUD works correctly on the wide range of 2005 PC
displays ([C58.1](../C58-Build-Pipeline/01-shipping-exe.md)) from 4:3 CRTs to 16:9 panels, without a combinatorial
explosion of per-display layouts. Understanding this split is understanding *why the HUD is always where it should
be*, on any screen.

## RE implications

- **Resolution** is solved by **normalized coords × backbuffer** — one layout, every resolution, no per-resolution
  data.
- **Aspect** is solved by **package selection** — `WIDESCREEN_GLOBAL` vs `THINSCREEN_GLOBAL` twins (+ `WS_*.fng`) —
  hand-tuned per aspect.
- **The live choice** is `FEngHud::mCurrentWidescreenSetting`.
- **The split** — normalize for size, select for aspect — covers all displays with two layouts, no combinatorial
  blowup.

---

### Key takeaways

- **Resolution** is handled by **normalization** ([C65.3](03-pursuit-hud.md)) — `pixel = normalized × backbuffer` —
  so **one layout serves every resolution** with no per-resolution data.
- **Aspect ratio** is handled by **whole-package selection** — **`WIDESCREEN_GLOBAL`** (16:9) vs
  **`THINSCREEN_GLOBAL`** (4:3) twin FE bundles, plus explicit `WS_*.fng` screens — each a **hand-tuned** layout,
  not a stretched one.
- The live choice is recorded on **`FEngHud::mCurrentWidescreenSetting`**.
- Package selection (not per-element math) means each aspect gets a **designed** layout, no runtime aspect
  computation — the **data-over-code** pattern.
- The **split** — normalize for size, select for aspect — covers **any resolution × 4:3/16:9** with just **two
  layouts**; editing the *wrong* aspect's package is a classic modding trap ([C65.8](08-reading-hud.md)).

**Continue:** [C65.5 — Gauges & meters](05-gauges-meters.md) · [Chapter 65 hub](C65-HUD-Runtime.md)
