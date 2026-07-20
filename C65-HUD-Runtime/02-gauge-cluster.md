# C65.2 — The HUD Object Model

> **The one-sentence version:** `IPlayer::GetHud()` returns a per-player `FEngHud` holding one named `HudElement*`
> pointer per widget (speedometer, heat meter, busted meter, minimap, pursuit board…); each `HudElement` holds a
> tree of `FEObject`s and a 64-bit feature mask, and gets `Update(IPlayer*)` every frame.

[← C65.1 — The FEng runtime](01-hud-runtime.md) · [Chapter 65 hub](C65-HUD-Runtime.md) ·
[Next: C65.3 — Placement & anchoring →](03-pursuit-hud.md)

---

## FEngHud: the per-player container

Each player has a **`FEngHud`** — reached via `IPlayer::GetHud()` — the container that owns every HUD widget as a
**named pointer**. The single most useful fact for HUD work: every meaningful widget is a distinct field on this
one struct:

```cpp
struct FEngHud /* : IHud */ {
  uint64_t     CurrentHudFeatures;   // bitmask of enabled HUD features (mode gating)
  ePlayerHudType mPlayerHudType;     // which HUD family (race/drag/split…)
  const char*  pPackageName;         // the bound .fng package
  IPlayer*     pPlayer;  int32_t PlayerNumber;
  bool         mInPursuit;           // flips the pursuit-widget masks as a group
  // — the widgets, each a HudElement* —
  HudElement*  pSpeedometer;   HudElement* pTachometer;   HudElement* pTachometerDrag;
  HudElement*  pHeatMeter;     HudElement* pBustedMeter;  HudElement* pGetAwayMeter;
  HudElement*  pNitrous;       HudElement* pSpeedBreakerMeter; HudElement* pEngineTemp;
  HudElement*  pLeaderBoard;   HudElement* pPursuitBoard; HudElement* pMilestoneBoard;
  HudElement*  pRaceInformation; HudElement* pInfractions; HudElement* pCostToState;
  HudElement*  pMinimap;       HudElement* pRadarDetector; HudElement* p321Go;
  HudElement*  pGenericMessage; HudElement* pRaceOverMessage; HudElement* pWrongWIndi;
  bool         mCurrentWidescreenSetting;  // 16:9 vs 4:3 (C65.4)
};
```

So the HUD is a *flat list of named widgets* per player — `pSpeedometer`, `pHeatMeter`, `pBustedMeter`, `pMinimap`,
`pPursuitBoard`, and so on. Split-screen ([C65.4](04-resolution-widescreen.md)) means *two* `FEngHud`s, one per
`IPlayer`.

> 🟡 *Reasoned (community-tooling-consistent, cross-checked against verified strings):* the `FEngHud` field layout
> is from front-end tooling structures; the widget *names* it references are byte-verified strings in `speed.exe`
> (`Hud_HeatMeter`, `Hud_BustedMeter`, `Hud_GetAwayMeter`, `Hud_MilestoneBoard`, `Hud_DragTachometer`, `321_GO`,
> `Speedometer`, `Tachometer`, …), and the per-player `GetHud()` accessor is consistent with the verified FEng
> runtime ([C65.1](01-hud-runtime.md)).

## HudElement: the widget base

Each widget is a **`HudElement`** — the base class holding its FEng objects, its mode mask, and its per-frame
update:

```cpp
struct HudElement {
  bPList<FEObject> Objects;            // the widget's FEng object tree (images/strings/groups)
  const char*      pPackageName;       // owning .fng package
  uint64_t         Mask;               // feature bits this element responds to
  uint64_t         CurrentHudFeatures; // mirror of the active feature set
  bool             mCurrentlySetVisible;
  virtual void     Update(IPlayer* player);  // called every frame while active
};
```

Three things matter:

- **`Objects`** — the widget's *contents*: a tree of `FEObject`s ([below](#feobject-the-widget-contents)) — the
  images, strings, and groups the runtime draws and positions.
- **`Mask`** — a 64-bit **feature mask** tested against the HUD's `CurrentHudFeatures` to decide visibility *per
  mode* ([C65.7](07-rendering-update.md)) — a widget shows iff its mask bits are in the active set.
- **`Update(IPlayer*)`** — the **per-frame hook**: while active, each element's `Update` runs every frame, pulling
  live values (speed, heat, …) into its objects ([C65.5](05-gauges-meters.md)). This is the sanctioned seam for
  reading or overriding a widget.

So a `HudElement` bundles *what it shows* (`Objects`), *when it shows* (`Mask`), and *how it refreshes*
(`Update`) — the three facets of a HUD widget.

> 🟡 *Reasoned:* the `HudElement` layout is community-tooling-consistent; its `Objects`/`Mask`/`Update` structure is
> consistent with the verified per-frame FE draw ([C65.7](07-rendering-update.md)) and the mode-gated widget
> strings.

## FEObject: the widget contents

Inside a `HudElement`'s `Objects` list are **`FEObject`s** — the FEng primitives the runtime actually manipulates:

- **`FEImage`** — one atlas-region quad (a gauge face, an icon) — references its texture by hash
  ([C65.3](03-pursuit-hud.md)), so re-skinning ([C65.5](05-gauges-meters.md)) doesn't move it.
- **`FEMultiImage`** — a multi-region image (e.g. a meter built from several atlas pieces).
- **strings/groups** — text (by label, [Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) and
  grouped sub-objects (`321_GO_GROUP`, `ENGINE_METER` group).
- **`FEVector2/3`** — the placement vectors ([C65.3](03-pursuit-hud.md)) — where each object sits.

So a widget is a *composite* of FEObjects — a speedometer is a backing image + a needle image + digit strings,
grouped. This composite structure is why a widget can be re-skinned (edit the atlas), re-positioned (edit the
placement vectors), or re-worded (edit the labels) *independently* ([C65.3](03-pursuit-hud.md)) — the three are
separate FEObject attributes.

## Why a named-pointer object model

The per-player, named-pointer model ([above](#fenghud-the-per-player-container)) is what makes the HUD *tractable*:

- **Every widget is directly reachable.** `player->GetHud()->pHeatMeter` *is* the heat meter — hiding, inspecting,
  or repositioning it starts from one pointer ([C65.8](08-reading-hud.md)). No searching a scene graph.
- **Per-player isolation.** Split-screen ([C65.4](04-resolution-widescreen.md)) gives each `IPlayer` its own
  `FEngHud` — the two HUDs are independent, updated separately.
- **Uniform update.** Every active widget ticks the same way — `Update(IPlayer*)` — a uniform per-frame refresh
  ([C65.5](05-gauges-meters.md)).
- **Mode gating in data.** The feature masks ([C65.7](07-rendering-update.md)) decide visibility, so the *same*
  widget set reconfigures per mode without code — flip `mInPursuit` and the pursuit widgets appear.

So the object model is a *flat, named, per-player* set of widgets, each a self-refreshing composite of FEObjects,
gated by masks. It's the runtime face of the FEng packages ([C65.1](01-hud-runtime.md)) — the package defines the
widgets, the object model exposes them as reachable pointers. Reading it ([C65.8](08-reading-hud.md)) is the entry
to everything HUD: the pointers are where you inspect, hide, move, or override.

## RE implications

- **`FEngHud`** (per player, via `IPlayer::GetHud()`) holds a **named `HudElement*` per widget** — the reachable
  entry point.
- **`HudElement`** = `Objects` (FEObject tree) + `Mask` (mode gating) + `Update(IPlayer*)` (per-frame refresh).
- **`FEObject`/`FEImage`** are the contents — atlas quads (by hash), strings (by label), groups, placement vectors.
- **Named-pointer model** — directly reachable widgets, per-player isolation, uniform update, data mode-gating.

---

### Key takeaways

- Each player has a **`FEngHud`** (via `IPlayer::GetHud()`) holding **one named `HudElement*` per widget** —
  `pSpeedometer`, `pHeatMeter`, `pBustedMeter`, `pMinimap`, `pPursuitBoard`, … — the reachable entry to every HUD
  element.
- A **`HudElement`** bundles **`Objects`** (its FEObject tree), a **`Mask`** (feature bits for per-mode visibility),
  and **`Update(IPlayer*)`** (the per-frame refresh hook).
- Widget contents are **`FEObject`s** — `FEImage` atlas quads (referenced **by hash**), strings (**by label**),
  groups, and placement vectors — so art, position, and text edit **independently**.
- The **named-pointer, per-player** model makes the HUD tractable — every widget directly reachable, split-screen
  isolated, uniformly updated, mode-gated in data.
- The object model is the **runtime face of the FEng packages** — the packages define the widgets, the model
  exposes them as pointers ([C65.8](08-reading-hud.md)).

**Continue:** [C65.3 — Placement & anchoring](03-pursuit-hud.md) · [Chapter 65 hub](C65-HUD-Runtime.md)
