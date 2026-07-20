# C65.7 — Rendering & Update

> **The one-sentence version:** the HUD is drawn last in the frame — after the entire post-processing stack, before
> `Present` — so it stays sharp and unblurred; it updates three ways: per-frame value bindings, feature masks
> (mode switches), and discrete `EShow*`/`M*` events routed through `GEventTable::Post` and the Lua flow.

[← C65.6 — Scoreboards](06-scoreboards.md) · [Chapter 65 hub](C65-HUD-Runtime.md) ·
[Next: C65.8 — Reading the HUD in RE →](08-reading-hud.md)

---

## Drawn last: unblurred by design

The HUD is drawn **last in the frame** — after the *entire* render and post-processing stack
([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)), immediately before `Present`:

```
world visibility → geometry → road → props → vehicles → shadows → reflections
   → particles → speed FX → bloom → motion blur → HUD → Present
```

**Why last:** the HUD must **not** be bloomed, blurred, or color-graded — UI has to read *cleanly* at 200 mph.
Drawing it *after* the post stack ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) is what keeps the
speedometer and minimap **sharp while the world smears** with motion blur. If the HUD were drawn *before* the post
stack, the bloom and blur would wash it out — an unreadable HUD. So the render order is deliberate: post-process the
*world*, then stamp the *crisp* HUD on top. This also defines the modding boundary
([C65.8](08-reading-hud.md)): a D3D9 `Present`/`EndScene` hook lands *after* the HUD, so the HUD is already drawn by
then.

> 🟡 *Reasoned (model) / ✅ (anchors):* the pass ordering (HUD after the post stack, before `Present`) is the frame
> model consistent with the verified render pipeline ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md))
> and the unblurred-HUD requirement; the FE draw itself is verified (the `FERenderEPolySlotPoolOverflow` diagnostic,
> [C65.1](01-hud-runtime.md)).

## The FE draw path

The HUD draw is **screen-space quad submission** through FEng's element layer
([Chapter 51](../C51-Render-Pipeline/03-effect-system.md)):

- **Each visible `FEObject`** ([C65.2](02-gauge-cluster.md)) resolves its **atlas texture by hash**, its
  **rectangle by layout** ([C65.3](03-pursuit-hud.md), normalized × backbuffer), and submits a quad.
- **Text** resolves label → glyph quads from the font atlas
  ([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) — fonts are the *same* atlas+table pattern as the HUD.
- **The FE polygon slot pool** holds the submitted quads — overflow guarded by the verified
  **`FERenderEPolySlotPoolOverflow`** diagnostic ([C65.1](01-hud-runtime.md)), a pooled allocator
  ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) like the rest of the engine.

So the HUD is a stream of textured 2D quads (atlas regions and glyphs), positioned by the normalized layouts
([C65.3](03-pursuit-hud.md)), pooled and drawn in one pass. It's the *2D* counterpart of the 3D render
([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)) — same device, same pooling, screen-space instead of
world-space.

> ✅ *Verified:* `FERenderEPolySlotPoolOverflow` is present in `speed.exe` — the FE render poly-slot pool
> diagnostic.

## Three update mechanisms

The HUD changes via **three distinct mechanisms** at three timescales:

**1. Per-frame value bindings (continuous).** Every active `HudElement::Update(IPlayer*)`
([C65.5](05-gauges-meters.md)) pulls live values each frame — speed, RPM, heat, the bust meter. The gauges and
meters move continuously off the `FrameTick` spine ([C37.4](../C37-Frame-Spine-Modules/04-frametick.md)).

**2. Feature masks (mode switches).** The HUD's `CurrentHudFeatures` bitmask is tested against each element's
`Mask` ([C65.2](02-gauge-cluster.md)) to decide visibility *per mode*. Setting **`FEngHud::mInPursuit` flips the
pursuit group at once** — heat/busted/pursuit-board on, race-only widgets off. This is why the HUD "reconfigures
itself" *without reloading* — same package, different feature bits.

**3. Discrete events (`EShow*`/`M*`).** One-time changes ride the engine's messaging
([C65.6](06-scoreboards.md)):

- **`EShow*` events** (verified Kinds, lookup2 of the name): `EShowRaceCountdown`, `EShowRaceOverMessage`,
  `EShowResults`, `EShowSMS`, `EShowMilestones`, …
- **`M*` messages**: `MCountdownDone`, `MNotifyFinished`, `MNotifyRacePlacement`, `MPerpBusted`/`MPerpEscaped`,
  `MEnterPostRaceFlow`, … — dispatched through **`GEventTable::Post` (`0x65FAF0`)** and consumed by FE controllers
  and the Lua stategraph ([C65.1](01-hud-runtime.md)).

So the HUD updates *continuously* (value bindings), *by mode* (feature masks), and *on events* (`EShow*`/`M*`) —
three mechanisms for three kinds of change (a moving needle, a mode switch, a one-time overlay). Reacting to the
HUD *correctly* means using the right one: poll nothing — bind values, gate by mask, listen for events.

> ✅ *Verified:* `GEventTable::Post` is at `0x65FAF0`; the `EShow*`/`M*` vocabulary (e.g. `MPerpBusted`/
> `MPerpEscaped`, [C48.4](../C48-Pursuit-Heat/04-bust-evade.md)) is present. The feature-mask mechanism is
> 🟡 community-tooling-consistent, aligned with the verified mode strings.

## The GameFlow tie-in

Discrete HUD/screen changes are sequenced by **GameFlow** ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md))
— the phase machine ([C54.1](../C54-GameFlow-Blacklist/01-gameflow-states.md)) that drives the front-end flow:

- **FE phases** — `GameFlowLoadingFrontEndPart1/2`, `LoadingFrontEnd` — load the FE/HUD packages
  ([C65.1](01-hud-runtime.md)).
- **Load flavors** — `eLOAD_GeneralLoading` vs **`eLOAD_BustedLoading`** — the busted loading screen is its *own*
  state ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)).
- **Career glue** — `SetRaceCompleteForFE` (`0x604C90`), `StorePursuitRepForFE` (`0x604CA0`) — write results for
  the FE result screens ([C65.6](06-scoreboards.md)) to display.

So a race ending ([C55.1](../C55-Race-Events/01-race-flow.md)) is: the sim posts `MEnterPostRaceFlow`, GameFlow
transitions to the post-race phase, the career glue writes the results, and the FE loads and shows the result
screen ([C65.6](06-scoreboards.md)). The HUD's *discrete* changes (overlays, screens) are thus orchestrated by the
event/flow layer ([above](#three-update-mechanisms)), while its *continuous* changes (gauges) run off the frame
spine. Together — value bindings, masks, events, flow — they are *everything* that makes the HUD change. Reading
them ([C65.8](08-reading-hud.md)) tells you exactly *why* any HUD element appeared, moved, or updated.

## RE implications

- **Drawn last** (after the post stack, before `Present`) — unblurred by design; a `Present` hook lands after it.
- **The FE draw** — screen-space quads (atlas by hash, layout by normalized rect) through a pooled slot pool
  (`FERenderEPolySlotPoolOverflow`).
- **Three update mechanisms** — per-frame value bindings, feature masks (mode), `EShow*`/`M*` events (via
  `GEventTable::Post`).
- **GameFlow sequences** the discrete screens; career glue writes result data.

---

### Key takeaways

- The HUD is **drawn last** — after the entire post-processing stack, before `Present` — so it stays **sharp and
  unblurred** while the world smears (a `Present`/`EndScene` hook lands *after* it).
- The **FE draw** is screen-space quads — each `FEObject` resolves its **atlas by hash** and **rectangle by
  normalized layout**, pooled through the slot pool (`FERenderEPolySlotPoolOverflow`); text is glyph quads from the
  font atlas.
- The HUD updates **three ways**: **per-frame value bindings** (gauges/meters), **feature masks** (mode switches —
  `mInPursuit` flips the pursuit group without reloading), and **discrete `EShow*`/`M*` events** (via
  `GEventTable::Post 0x65FAF0`).
- **GameFlow** sequences the discrete screens (FE load phases, the `eLOAD_BustedLoading` state); career glue
  (`SetRaceCompleteForFE`/`StorePursuitRepForFE`) writes result data for the FE.
- To react to the HUD correctly: **bind values, gate by mask, listen for events** — don't poll.

**Continue:** [C65.8 — Reading the HUD in RE](08-reading-hud.md) · [Chapter 65 hub](C65-HUD-Runtime.md)
