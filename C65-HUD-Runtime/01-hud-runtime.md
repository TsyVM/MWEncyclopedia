# C65.1 — The FEng Runtime

> **The one-sentence version:** the HUD is FEng — the front-end engine that also draws the menus — running UI over
> gameplay: each screen is a `.fng` FEPackage (chunk `0x00030203`, magic `FEn`/`PkHd`) bound at load to a C++
> controller via a 178-row registry at `0x8F4320`, with flow logic in Lua 5.0.1.

[← Chapter 65 hub](C65-HUD-Runtime.md) · [Next: C65.2 — The HUD object model →](02-gauge-cluster.md)

---

## The HUD is front-end UI

The single organising fact of the MW05 HUD is that it is **not a hard-coded overlay** — it is **FEng** (the
FrontEnd Engine), the *same* UI runtime that draws the main menu, garage, and map, drawing over the running game.
The speedometer is a front-end widget on top of gameplay, built exactly like a menu button. This means the HUD
inherits the engine-wide UI idiom ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)):

- **Pixels** live in TPK texture atlases ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) — `HUDTEXRACE`,
  `HUDS_Custom_*` ([C65.5](05-gauges-meters.md)).
- **Positioning** lives in layout records ([C65.3](03-pursuit-hud.md)) — normalized screen coordinates.
- **Text** comes by label ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) — language-neutral.

So "the HUD" and "the menus" are one system with two uses, and everything about the HUD follows from understanding
FEng.

> ✅ *Verified:* FEng is a real module in `speed.exe` — `FERenderEPolySlotPoolOverflow` (the FE render poly-slot
> diagnostic), the HUD packages `HUD_SingleRace`/`HUD_Drag`/`HUD_Player1`, the overlays `321_GO`/`BUSTED_OVERLAY`/
> `FLASHERS`, `CustomHUD`/`CustomHUDColor`, the middleware tag `EAGL4` (×8), and `EVENT_FENG` are all present.

## The `.fng` FEPackage

An `.fng` "file" is an **EAGL chunk** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) inside a
bundle — chunk ID **`0x00030203`**, carrying an **FEPackage**:

- **Magic `FEn`/`PkHd`** — verified present (`FEn` ×15, `PkHd` ×5); `.fng` appears **215×** in the exe (the
  screen-name vocabulary).
- **The package name at `pkg+0x28`** — `FEPackage::BindController` (`0x585520`) reads it, hashes it
  ([below](#the-screen-registry)), and finds the controller.
- **The interior** (widget tree of images/strings/groups, per-object animation scripts, transitions) is the
  package payload — bracketed by the `FEn`/`PkHd` magic and the controller binding, and parseable by community FEng
  tooling ([C65.8](08-reading-hud.md)); the encyclopedia treats the exact interior byte-format as an open edge
  ([C65.8](08-reading-hud.md)).

So each HUD screen (`HUD_SingleRace.fng`, `BUSTED_OVERLAY.fng`, `321_GO`) is a data package
([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) loaded like any asset — the HUD's *content* is
data, its *behaviour* is the controller ([below](#controllers)) + Lua ([below](#the-lua-flow-layer)).

> ✅ *Verified:* `FEn`/`PkHd` and `.fng` (×215) are present; `FEPackage::BindController` at `0x585520` is a real
> function (`53 8B 5C 24 08 …` = `push ebx; mov ebx,[esp+8]`).

## The screen registry

The exe holds a static **registry of 178 rows `{fngNameHash, controllerFactory}` at `0x8F4320`** — byte-confirmed
by the code that reads it:

- **`FEScreen_FindRow` (`0x571DD0`)** — `(fngNameHash) → row*` — its prologue `56 33 F6 8B 04 F5 20 …` is
  `push esi; xor esi,esi; mov eax,[esi*8 + 0x8F4320]` — a **scan of 8-byte rows based at `0x8F4320`**, directly
  confirming the table's address and stride.
- **`FEScreen_CreateController` (`0x571D60`)** — `(fngNameHash,…)` finds the row and calls its factory.
- **`FngNameHash` (`0x5AF1C0`)** — the lookup2 hash of the `.fng` name (the registry key); its prologue
  `8B 54 24 04 83 C8 FF` (`mov edx,[esp+4]; or eax,-1`) is a hash init.
- **`FE::RegisterInGameScreens` (`0x4C2340`)** — gated on `[0x911FA8]` (its prologue `A1 A8 1F 91 00` =
  `mov eax,[0x911FA8]`) — registers the in-game/pause screen set *only while the world is active*.

So loading a `.fng` is: read its name → `FngNameHash` → `FEScreen_FindRow` in the `0x8F4320` table → call the
factory to build the controller. This registry *is* the map from screen names to code, and it's verified down to
the table address and stride.

> ✅ *Verified:* the registry is at `0x8F4320` (8-byte rows), confirmed by `FEScreen_FindRow`'s `mov eax,[esi*8+0x8F4320]`;
> `FngNameHash` (`0x5AF1C0`), `FEScreen_CreateController` (`0x571D60`), and `FE::RegisterInGameScreens` (`0x4C2340`,
> gate `[0x911FA8]`) are real functions.

## Controllers

Each screen gets a small C++ **controller** ("view") class — its constructor installs a unique vtable
([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) and binds to its package by name. For the HUD/world,
`FEWorldMapScreen::ctor` (`0x59D500`, an SEH prologue `6A FF 68 F3 2F 87 00`) is the map screen that consumes the
`HELICOPTER_LINE_OF_SIGHT` overlay ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)). The controller's job is pure UI
plumbing — reads from the FE attribute vault, posts events (`GEventTable::Post`,
[C65.7](07-rendering-update.md)), and drives the package's widgets. Screen→screen *transitions* are data-driven
*inside* the packages, not a code matrix.

> ✅ *Verified:* `FEWorldMapScreen::ctor` at `0x59D500` is a real function (SEH prologue); `HELICOPTER_LINE_OF_SIGHT`
> is present.

## The Lua flow layer

Crucially, the front-end **flow and menu logic is Lua 5.0.1 script** (the version string is in the exe), not
hard-coded C++. The engine calls into script at named points (loading, race-over, bust-load, SMS, …) and scripts
react to the `M*`/`E*` message vocabulary ([C65.7](07-rendering-update.md)). The practical consequence for the HUD:
*behaviour* changes are often **script/data changes**, and patching C++ for menu/flow logic misses where the logic
actually lives. This is the same data/script-over-code philosophy as the rest of the engine
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) — the front-end is *authored*, not *compiled*.

> ✅ *Verified:* `Lua 5.0.1` is present in `speed.exe` — the front-end scripting layer; `Game failed to start.` is
> the FE fail-fast string when its data is missing.

## RE implications

- **The HUD is FEng** — front-end UI over gameplay; pixels in atlases, positioning in layouts, text by label.
- **`.fng` FEPackages** (chunk `0x00030203`, `FEn`/`PkHd`) are the screens; bound to controllers via the
  **`0x8F4320` registry** (verified address/stride).
- **`FngNameHash` (lookup2) → `FEScreen_FindRow` → factory** is the name→code path.
- **Flow is Lua 5.0.1** — HUD behaviour is often script/data, not C++.

---

### Key takeaways

- The HUD **is FEng** — the front-end engine that also draws the menus — so the speedometer is built like a menu
  button: **pixels in atlases, positioning in layouts, text by label**.
- Each screen is a **`.fng` FEPackage** (chunk `0x00030203`, magic `FEn`/`PkHd`; `.fng` ×215) bound at load to a
  C++ **controller**.
- The **178-row registry at `0x8F4320`** maps `.fng` name-hashes to controller factories — **byte-confirmed** by
  `FEScreen_FindRow`'s `mov eax,[esi*8+0x8F4320]`.
- Names are keyed by **`FngNameHash`** (`0x5AF1C0`, lookup2); in-game screens register via `FE::RegisterInGameScreens`
  only while the world is active (gate `[0x911FA8]`).
- **Flow logic is Lua 5.0.1** — HUD/menu *behaviour* is script/data, not compiled C++ (patch the script, not the
  binary).

**Continue:** [C65.2 — The HUD object model](02-gauge-cluster.md) · [Chapter 65 hub](C65-HUD-Runtime.md)
