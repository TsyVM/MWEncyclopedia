# Chapter 65 — The In-Race HUD & FEng: Placement, Scoreboards & the On-Screen Runtime

> **Goal of this chapter:** decode the in-race HUD as what it really is — **front-end UI (FEng) running over
> gameplay**. The same FEng screen system that draws the main menu draws the speedometer. This chapter covers the
> FEng runtime (`.fng` packages, the screen registry, controllers), the per-player HUD object model
> (`FEngHud`→`HudElement`→`FEObject`), how **placement/anchoring** works (normalized screen space, so one layout
> serves every resolution), the widescreen/thinscreen mechanism, the gauges and meters, the **scoreboards**
> (leaderboard, pursuit board, milestones), where the HUD renders (last in the frame, unblurred), and exactly what
> makes it update.

The MW05 HUD is **not a hard-coded overlay** — it's the front-end engine (**FEng**) drawing UI over the running
game. This single fact organises everything: pixels live in TPK atlases, *positioning* lives in layout tables
(normalized `0..1` screen space), and text comes by label — the same engine-wide idiom as the menus
([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)). This chapter is the deep technical logic of that
system: the packages, the object model, the placement math, the scoreboards, the render position, and the update
paths.

> **Verified against the executable.** FEng is a real module in `speed.exe`: the render diagnostic
> **`FERenderEPolySlotPoolOverflow`**, the HUD packages **`HUD_SingleRace`/`HUD_Drag`/`HUD_Player1`**, the overlays
> **`321_GO`/`BUSTED_OVERLAY`/`FLASHERS`**, the config screens **`CustomHUD`/`CustomHUDColor`**, the package magic
> **`FEn`/`PkHd`** (`.fng` appears **215×**), the middleware tag **`EAGL4`** (×8), the front-end scripting
> **`Lua 5.0.1`**, and **`EVENT_FENG`** are all present. The screen registry is byte-confirmed: **`FEScreen_FindRow`
> (`0x571DD0`)** indexes a table with `mov eax,[esi*8 + 0x8F4320]` (8-byte rows at `0x8F4320`);
> **`FEScreen_CreateController` (`0x571D60`)**, **`FEPackage::BindController` (`0x585520`)**, **`FngNameHash`
> (`0x5AF1C0`, a lookup2)**, and **`FE::RegisterInGameScreens` (`0x4C2340`, gated on `[0x911FA8]`)** are all real
> functions. Aspect twins **`WIDESCREEN_GLOBAL`/`THINSCREEN`** (+10 `WS_*.fng` screens) are present. The
> object-model struct layouts (`FEngHud`/`HudElement`) are community-tooling-consistent (🟡), cross-checked against
> the verified widget strings.

---

## Deep-dive pages

- [C65.1 — The FEng runtime](01-hud-runtime.md): the front-end engine that owns the HUD (packages, registry,
  controllers, Lua).
- [C65.2 — The HUD object model](02-gauge-cluster.md): `FEngHud`→`HudElement`→`FEObject`, per-player named
  pointers, feature masks.
- [C65.3 — Placement & anchoring](03-pursuit-hud.md): normalized `0..1` screen space; the three independent axes.
- [C65.4 — Resolution & widescreen](04-resolution-widescreen.md): one layout, every resolution; the aspect twins.
- [C65.5 — Gauges & meters](05-gauges-meters.md): speedometer/tachometer, heat/busted/get-away — value bindings.
- [C65.6 — Scoreboards](06-scoreboards.md): leaderboard, pursuit board, milestones, race information.
- [C65.7 — Rendering & update](07-rendering-update.md): drawn last (unblurred); per-frame, mask, and event updates.
- [C65.8 — Reading the HUD in RE](08-reading-hud.md): navigation, modding recipes, and open edges.

---

## 65.1 The FEng runtime

The HUD is **FEng** — EA Black Box's front-end engine ([C65.1](01-hud-runtime.md)) — the *same* runtime that draws
menus, drawing UI over gameplay. Each screen is a **`.fng` FEPackage** (chunk `0x00030203`, magic `FEn`/`PkHd`),
bound at load to a C++ **controller** via a **178-row `{fngNameHash, factory}` registry at `0x8F4320`**
(byte-confirmed). Screen names are keyed by **`FngNameHash`** (`0x5AF1C0`, lookup2). Menu/flow *behaviour* is
**Lua 5.0.1** script — so HUD behaviour is often data/script, not C++.

## 65.2 The HUD object model

`IPlayer::GetHud()` returns a per-player **`FEngHud`** — one **named `HudElement*` pointer per widget**
([C65.2](02-gauge-cluster.md)): `pSpeedometer`, `pHeatMeter`, `pBustedMeter`, `pMinimap`, `pLeaderBoard`,
`pPursuitBoard`, `pMilestoneBoard`, … Each `HudElement` holds a tree of **`FEObject`s** (images/strings/groups) and
a 64-bit **feature mask** deciding its visibility per game mode. `Update(IPlayer*)` runs every frame — the natural
seam for reading or overriding a widget.

## 65.3 Placement & anchoring

Positioning follows the engine idiom: **atlas carries pixels, a layout record carries placement**
([C65.3](03-pursuit-hud.md)). The layout record holds an **`anchor` (vec2) and `size` (vec2) in *normalized*
`0..1` screen space** — `(0.5,0.5)` is center, `(1,1)` full extent. The renderer multiplies by the actual
backbuffer size, so *one layout serves every resolution*. Position, art (by hash), and text (by label) edit on
**three independent axes**.

## 65.4 Resolution & widescreen

Because anchors are **normalized** ([C65.3](03-pursuit-hud.md)), the HUD is resolution-independent
([C65.4](04-resolution-widescreen.md)) — the same fractions scale to 640×480 or 1920×1080. **Aspect ratio** is
handled by *package selection*, not per-element math: **`WIDESCREEN_GLOBAL`** vs **`THINSCREEN_GLOBAL`** are twin FE
bundles (16:9 vs 4:3), and loading screens ship explicit `WS_*.fng` twins. The live choice is
`FEngHud::mCurrentWidescreenSetting`.

## 65.5–65.6 Gauges, meters & scoreboards

The **gauges/meters** ([C65.5](05-gauges-meters.md)) are value-bound each frame — the speedometer from
`|velocity|` ([C41.5](../C41-Physics-RigidBody/05-integrate-math.md)), the tachometer from RPM, and the
**busted/get-away bars *are* the bust-envelope meter** ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md), `[perp+0x120]`).
The **scoreboards** ([C65.6](06-scoreboards.md)) — leaderboard, pursuit board, milestone board, race information —
are the *list/tabular* widgets (`COLUMN%d_DATA`/`LINE%d_GROUP`), populated from race and pursuit state.

## 65.7 Rendering & update

The HUD is drawn **last in the frame** (after the whole post-processing stack, before `Present`)
([C65.7](07-rendering-update.md)) — so it stays **sharp and unblurred** while the world smears; FE quads flow
through a poly slot pool (`FERenderEPolySlotPoolOverflow`). It updates three ways: **per-frame value bindings**
(speed, RPM, heat), **feature masks** (mode switches — `mInPursuit` flips the pursuit group at once), and discrete
**`EShow*`/`M*` events** routed through `GEventTable::Post` and the Lua flow.

---

### Key takeaways

- The HUD **is FEng** — the front-end engine drawing UI over gameplay; the speedometer and the main menu use the
  same system (packages, registry, controllers, Lua).
- Each widget is a **named `HudElement*` on the per-player `FEngHud`**, holding a tree of `FEObject`s and a
  **feature mask** for per-mode visibility.
- **Placement is normalized `0..1` screen space** (`anchor`/`size` × backbuffer) — so **one layout serves every
  resolution**; aspect is handled by **package selection** (`WIDESCREEN`/`THINSCREEN` twins), not per-element math.
- The **busted/get-away bars are the bust-envelope meter** ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)); the
  **scoreboards** (leaderboard/pursuit/milestone) are the list widgets.
- The HUD is drawn **last** (unblurred), and updates via **value bindings + feature masks + `EShow*`/`M*` events**.

**Next:** [Chapter 66 — Interactive Music & the Pursuit Score](../C66-Interactive-Music/C66-Interactive-Music.md):
the soundtrack that responds to the chase.
