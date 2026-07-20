# Chapter 66 — Interactive Music & the Pursuit Score

> **Goal of this chapter:** decode the music *runtime* (as opposed to the MUS/MPF data format of Chapter 21) — the
> `MusicFlow` state machine, the EA Trax licensed-song jukebox (`TraxMusicMode`, `Jukebox`), and the **interactive
> score** (`Interactive`, `InteractiveMusicVol`, `Intensity`) that intensifies with the pursuit, driven by the
> AI→audio bridge.

Chapter 21 decoded how music is *stored* (MUS/MPF, the routing graph); this chapter decodes how it's *played and
adapted*. Most Wanted has *two* music systems: the **EA Trax** jukebox of licensed songs (the menu/free-roam
soundtrack) and the **interactive score** that dynamically responds to gameplay — tensing as a pursuit escalates.
This chapter decodes both, and the `Intensity` mechanism that ties the score to the chase
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).

> **Verified against the executable.** The music runtime is named in `speed.exe`: **`MusicFlow`** (the music state
> machine), `MusicMode`, `MusicVol`; **EA Trax** — `EA_Trax`, `PFEATrax` (the music bus,
> [C59.1](../C59-Audio-Runtime/01-audio-runtime.md)), `Trax`/`TraxInit`/`TraxMusicMode`/`TraxScreen`, `Jukebox`; and
> the **interactive score** — `Interactive`, `InteractiveDone`, **`InteractiveMusicVol`**, **`Intensity`**. The
> AI→audio bridge is `SoundAI` ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)).

---

## Deep-dive pages

- [C66.1 — The music runtime](01-music-runtime.md): `MusicFlow` and the two music systems.
- [C66.2 — EA Trax](02-ea-trax.md): the licensed-song jukebox.
- [C66.3 — The interactive score](03-interactive-score.md): the dynamic gameplay music.
- [C66.4 — Intensity & Heat](04-intensity-heat.md): the score responding to the chase.
- [C66.5 — Reading music in RE](05-reading-music.md): navigating the music runtime.

---

## 66.1 The music runtime

The music runtime ([C66.1](01-music-runtime.md)) is governed by **`MusicFlow`** — the state machine that selects
*what music plays when*: menu music, free-roam Trax, race music, the pursuit score. It manages *two* systems: **EA
Trax** (licensed songs, [C66.2](02-ea-trax.md)) and the **interactive score** (dynamic gameplay music,
[C66.3](03-interactive-score.md)), each with its own volume (`MusicVol`, `InteractiveMusicVol`). Both route through
the `PFEATrax` music bus ([C59.1](../C59-Audio-Runtime/01-audio-runtime.md)).

## 66.2 EA Trax

**EA Trax** ([C66.2](02-ea-trax.md)) is the *licensed soundtrack* — the branded jukebox of songs (`TraxMusicMode`,
`TraxScreen` for song selection, `Jukebox`). It plays the menu and free-roam music — full songs, player-selectable.
This is the *branded* music of the mid-2000s NFS games ([C58.2](../C58-Build-Pipeline/02-eagl-engine.md)), a
marketing feature as much as an audio one. `EA_Trax` is initialised by `TraxInit`.

## 66.3 The interactive score

The **interactive score** ([C66.3](03-interactive-score.md)) is the *dynamic* music — the gameplay soundtrack that
*responds* to what's happening. Unlike a fixed song, the interactive score (`Interactive`, `InteractiveMusicVol`)
adapts its **`Intensity`** ([C66.4](04-intensity-heat.md)) to the game state — building tension in a pursuit,
easing when clear. It's the MUS/MPF routing graph ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) in
action: segments and transitions assembled live to match the moment.

## 66.4 Intensity & Heat

The key mechanism is **`Intensity`** ([C66.4](04-intensity-heat.md)) — a scalar that drives the interactive score's
tension, mapped from the game state. In a pursuit ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), Heat and
the bust envelope ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)) raise the intensity — the music grows *urgent* as
you're cornered, *triumphant* as you escape. This is driven by **`SoundAI`** ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md))
— the fleet→audio bridge that translates pursuit state into music intensity. The score is the *emotional* readout
of the chase, as the busted meter ([C65.5](../C65-HUD-Runtime/05-gauges-meters.md)) is the visual one.

---

### Key takeaways

- The music runtime is governed by **`MusicFlow`** (the state machine) and splits into **two systems** — **EA
  Trax** (licensed songs) and the **interactive score** (dynamic gameplay music) — both on the `PFEATrax` bus.
- **EA Trax** is the branded **jukebox** — full licensed songs, player-selectable (`TraxScreen`/`Jukebox`), for
  menus and free-roam.
- The **interactive score** (`Interactive`/`InteractiveMusicVol`) is **dynamic** — it adapts its **`Intensity`** to
  the game state, assembling MUS/MPF segments ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) live.
- **`Intensity`** maps from the pursuit — Heat and the bust envelope raise it (urgent when cornered, triumphant on
  escape) — driven by **`SoundAI`** ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)).
- The interactive score is the **emotional readout** of the chase — the aural counterpart of the busted meter.

**Next:** [Chapter 67 — Debug & Development Facilities](../C67-Debug-Facilities/C67-Debug-Facilities.md): the
developer tools left in the shipped game.
