# C65.8 — Reading the HUD in RE

> **The one-sentence version:** navigate the HUD from the `0x8F4320` screen registry down to the per-player
> `FEngHud` widget pointers — hide/inspect/move widgets at runtime via those pointers and `HudElement::Update`,
> re-skin via the `HUDTEX*` atlases, and mind the traps (wrong aspect twin, normalized ≠ pixels, behaviour in Lua).

[← C65.7 — Rendering & update](07-rendering-update.md) · [Chapter 65 hub](C65-HUD-Runtime.md) ·
[Next: Chapter 66 — Interactive Music & the Pursuit Score →](../C66-Interactive-Music/C66-Interactive-Music.md)

---

## Anchors for HUD RE

The HUD is anchored on verified structures and strings:

- **The FEng runtime** — the `0x8F4320` registry, `FngNameHash` (`0x5AF1C0`), `FEScreen_FindRow` (`0x571DD0`),
  `FEPackage::BindController` (`0x585520`) ([C65.1](01-hud-runtime.md)).
- **The object model** — `IPlayer::GetHud()` → `FEngHud` → the named `HudElement*` pointers
  ([C65.2](02-gauge-cluster.md)).
- **The placement** — normalized `anchor`/`size` layout records ([C65.3](03-pursuit-hud.md)); the
  `WIDESCREEN`/`THINSCREEN` twins ([C65.4](04-resolution-widescreen.md)).
- **The atlases** — `HUDTEXRACE`/`HUDTEXDRAG`/`HUDTEXSPLIT`/`HUDS_Custom_*` ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)).
- **The events** — `EShow*`/`M*` via `GEventTable::Post` (`0x65FAF0`) ([C65.7](07-rendering-update.md)).

From these, the whole HUD is navigable: the runtime, the widgets, the placement, the art, and the events.

## The RE workflow

Reading the HUD:

1. **Start at the registry** — the `0x8F4320` table ([C65.1](01-hud-runtime.md)); map `.fng` names to controllers.
2. **Reach the widgets** — `player->GetHud()` → the `FEngHud` pointers ([C65.2](02-gauge-cluster.md)); each widget
   is one pointer.
3. **Read placement** — the normalized layout records ([C65.3](03-pursuit-hud.md)); check the aspect twin
   ([C65.4](04-resolution-widescreen.md)).
4. **Trace updates** — value bindings, masks, and `EShow*`/`M*` events ([C65.7](07-rendering-update.md)).

The output is the full HUD picture: runtime, widgets, placement, art, and updates.

## Modding recipes

The HUD is unusually moddable *because* it's data-driven FEng ([C65.1](01-hud-runtime.md)) with reachable widget
pointers ([C65.2](02-gauge-cluster.md)):

| Goal | Route |
|---|---|
| **Hide the whole HUD** | the global HUD-draw toggle (a clamped variable write) |
| **Hide one widget** | `((FEngHud*)player->GetHud())->pHeatMeter->mCurrentlySetVisible = false;` |
| **Override widget behavior** | detour that widget's `HudElement::Update(IPlayer*)` — per-frame, per-widget, the sanctioned seam |
| **Re-skin gauges** | edit the `HUDTEX*`/`HUDS_Custom_NN` TPK pixels — **same size + format** ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) |
| **Restyle busted/countdown overlays** | edit the FEPackages in `InGameB.lzc` (affects NIS too — they composite the same overlays) |
| **Move/resize widgets** | runtime via the `FEObject` placement vectors today; on-disk layout writes wait on the layout chunk-ID confirmation ([below](#open-edges)) |
| **Change HUD text** | the label/string tables ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) — never literals in packages |
| **React to HUD moments** | listen for the `EShow*`/`M*` Kinds ([C65.7](07-rendering-update.md)) — don't poll |
| **Post-HUD screen effects** | a D3D9 `Present`/`EndScene` hook — lands *after* the HUD is drawn ([C65.7](07-rendering-update.md)) |

So the modding surface is layered: *runtime* (widget pointers — safe, immediate), *data* (atlases and packages —
durable), and *events* (the `EShow*`/`M*` vocabulary — the right way to react). The widget-pointer model
([C65.2](02-gauge-cluster.md)) makes runtime HUD edits precise — one pointer per widget.

> 🟡 *Reasoned:* the runtime modding routes (widget pointers, `HudElement::Update` detour, `mCurrentlySetVisible`)
> are community-tooling-consistent, resting on the verified FEng object model; the data routes (atlas/package edits,
> labels) rest on the verified asset formats ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[6](../C6-Texture-Codecs/C6-Texture-Codecs.md),
> [30](../C30-Localization-Labels/C30-Localization-Labels.md)).

## Known traps

The HUD's data-driven, resolution-independent design creates specific traps ([C65.3](03-pursuit-hud.md)–[C65.4](04-resolution-widescreen.md)):

- **Editing the wrong aspect twin** ([C65.4](04-resolution-widescreen.md)) — a HUD change that "doesn't show up" is
  often edited into the 4:3 package while you're playing 16:9 (or vice versa).
- **Editing the wrong mode's atlas** ([C65.5](05-gauges-meters.md)) — race art is in `HUDTEXRACE`, drag in
  `HUDTEXDRAG`, split-screen in `*SPLIT`, the custom skin in `HUDS_Custom_NN`.
- **Treating normalized anchors as pixels** ([C65.3](03-pursuit-hud.md)) — collapses the HUD into the corner.
- **Feeding world coordinates to minimap items** — `MiniMapItem::ItemPosition` is *minimap space*, not world space;
  the runtime projects world markers into it.
- **Patching C++ for behaviour that lives in Lua** ([C65.1](01-hud-runtime.md)) — menu/flow logic is script.
- **Expecting the garage performance bars to change physics** ([C65.5](05-gauges-meters.md)) — they're display
  normalizations, not the sim.

So HUD RE rewards knowing *where each thing lives*: position in normalized layouts, art in mode/aspect-specific
atlases, behaviour in Lua and `Update`, values in the sim. The traps are all *"edited the right kind of thing in
the wrong place"* — the fix is the layered model ([above](#the-re-workflow)).

## Open edges

Honesty about what remains unclosed ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)):

- **The `.fng` interior byte format** — object records, script tracks, transition tables — is bracketed by the
  `FEn`/`PkHd` magic and the controller binding ([C65.1](01-hud-runtime.md)) and parseable by community FEng
  tooling, but not byte-mapped in this encyclopedia's own corpus (🟡).
- **The HUD layout chunk ID** — the ~32-byte anchor/size record ([C65.3](03-pursuit-hud.md)) is decoded
  heuristically; confirming the chunk ID unlocks *trusted on-disk repositioning* (a high-value open contribution).
- **The exact `CurrentHudFeatures` bit assignments** — the feature-mask *mechanism* ([C65.7](07-rendering-update.md))
  is solid, but the bit-to-mode table is unenumerated.
- **The `FEngHud`/`HudElement` field offsets** are community-tooling-consistent (🟡), cross-checked against the
  verified widget strings, not independently byte-verified here.

So the HUD is *mostly* legible — the runtime, registry, object model, placement model, atlases, and events are
verified or well-grounded — with a few honest edges (the FNG interior, the layout chunk ID, the feature bits). This
is the verification-first discipline ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md))
applied to the HUD: verified where the bytes allow (the registry address, the function prologues, the strings),
reasoned where they stop (the struct offsets, the interior format), and open where neither reaches (the layout
chunk ID). Reading the HUD is reading FEng — and FEng is one of the most *self-documenting* systems in the game
(named screens, named widgets, named events), which is why so much of it yields to the strings.

## RE implications

- **Anchor on** the `0x8F4320` registry, the `FEngHud` widget pointers, the normalized layouts, the `HUDTEX*`
  atlases, and the `EShow*`/`M*` events.
- **The RE workflow** — registry → widgets → placement → updates.
- **Modding is layered** — runtime (pointers), data (atlases/packages/labels), events — the widget-pointer model
  makes it precise.
- **The traps** are "right thing, wrong place" — wrong aspect/mode, normalized-as-pixels, behaviour-in-Lua.

---

### Key takeaways

- The HUD is anchored on the **`0x8F4320` screen registry**, the per-player **`FEngHud` widget pointers**, the
  **normalized layouts**, the **`HUDTEX*` atlases**, and the **`EShow*`/`M*` events**.
- The RE workflow: **registry → widgets (`GetHud()` pointers) → placement (normalized, check the aspect twin) →
  updates (bindings/masks/events)**.
- **Modding is layered** — *runtime* (widget pointers: hide/inspect/detour `Update`), *data* (atlas/package/label
  edits), *events* (listen for `EShow*`/`M*`) — the widget-pointer model makes runtime edits precise.
- **Known traps** are all "right thing, wrong place" — the **wrong aspect twin**, the **wrong mode's atlas**,
  **normalized-as-pixels**, **world-coords into minimap items**, **C++ for Lua behaviour**, **bars ≠ physics**.
- **Open edges** (honest): the **`.fng` interior format**, the **layout chunk ID** (blocks trusted on-disk moves),
  the **feature-bit table**, and the **struct offsets** (🟡) — the rest is verified or well-grounded; FEng is
  self-documenting.

**Next:** [Chapter 66 — Interactive Music & the Pursuit Score](../C66-Interactive-Music/C66-Interactive-Music.md):
the soundtrack that responds to the chase.

**Sources:** `speed.exe` (verified: `FERenderEPolySlotPoolOverflow`; HUD packages `HUD_SingleRace`/`HUD_Drag`/
`HUD_Player1`; overlays `321_GO`/`BUSTED_OVERLAY`/`FLASHERS`; `CustomHUD`/`CustomHUDColor`; magic `FEn`/`PkHd`,
`.fng` ×215; `EAGL4`; `Lua 5.0.1`; `EVENT_FENG`; registry `0x8F4320` via `FEScreen_FindRow 0x571DD0`
`mov eax,[esi*8+0x8F4320]`; `FngNameHash 0x5AF1C0`, `FEScreen_CreateController 0x571D60`, `FEPackage::BindController
0x585520`, `FE::RegisterInGameScreens 0x4C2340` gate `[0x911FA8]`, `FEWorldMapScreen::ctor 0x59D500`,
`GEventTable::Post 0x65FAF0`; `WIDESCREEN_GLOBAL`/`THINSCREEN` + 10 `WS_*.fng`; widgets `Hud_HeatMeter`/`Hud_BustedMeter`/
`Hud_GetAwayMeter`/`Hud_LeaderBoard`/`PursuitBoard`/`Hud_MilestoneBoard`/`RaceInformation`/`Speedometer`/`Tachometer`/
`Hud_DragTachometer`; list bindings `COLUMN1/2/3_DATA`/`LINE%d_GROUP`; post screens `PostRace_Results`/`PostRace_Pursuit`/
`PostBusted`). Object-model struct layouts are 🟡 community-tooling-consistent, cross-checked against these strings.
The bust-envelope constants are byte-verified in [C48.4](../C48-Pursuit-Heat/04-bust-evade.md).
