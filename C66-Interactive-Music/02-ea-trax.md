# C66.2 — EA Trax

> **The one-sentence version:** EA Trax is the licensed-song jukebox — the branded soundtrack of full songs
> (`TraxMusicMode`, `TraxScreen`, `Jukebox`) that plays in menus and free-roam, player-selectable, initialised by
> `TraxInit`.

[← C66.1 — The music runtime](01-music-runtime.md) · [Chapter 66 hub](C66-Interactive-Music.md) ·
[Next: C66.3 — The interactive score →](03-interactive-score.md)

---

## The licensed jukebox

**EA Trax** is Most Wanted's *licensed soundtrack* system — the branded jukebox of full songs that defines the
game's music identity ([C58.2](../C58-Build-Pipeline/02-eagl-engine.md)):

- **Full licensed songs** — the mid-2000s rock/hip-hop/electronic tracks that were a marquee feature of the NFS
  series.
- **`TraxScreen`** — the song-selection UI ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) — the
  player picks tracks.
- **`Jukebox`** — the jukebox that plays the selected/enabled songs.
- **`TraxMusicMode`** — the Trax playback mode; `TraxInit` initialises the system.

So EA Trax is a *conventional* music player — a playlist of songs, selectable, played in sequence or shuffle. It's
the *branded* half of the music ([C66.1](01-music-runtime.md)), the licensed soundtrack the game was marketed with.
"EA Trax" was EA's soundtrack brand across many titles ([C58.2](../C58-Build-Pipeline/02-eagl-engine.md)) — a
recognisable feature.

> ✅ *Verified:* `EA_Trax`, `Trax`, `TraxInit`, `TraxMusicMode`, `TraxScreen`, `Jukebox`, and `PFEATrax` (the bus,
> [C59.1](../C59-Audio-Runtime/01-audio-runtime.md)) are present in `speed.exe` — the EA Trax system.

## Where Trax plays

EA Trax is the soundtrack of the *non-tense* contexts ([C66.1](01-music-runtime.md)):

- **Menus** — the front-end ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) plays Trax — the
  garage, the map, the Blacklist screens have a licensed-song backdrop.
- **Free-roam** — cruising the city (not in a pursuit) plays Trax — the ambient soundtrack of exploration.
- **Races** — races may play Trax (energetic songs) rather than the score, depending on the mode
  ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)).

So Trax is the *default* music — the licensed songs playing whenever the *interactive score*
([C66.3](03-interactive-score.md)) isn't needed (i.e. when there's no dynamic tension to score). The `MusicFlow`
state machine ([C66.1](01-music-runtime.md)) plays Trax in these contexts and *switches* to the interactive score
when a pursuit begins ([C66.4](04-intensity-heat.md)). So the player mostly hears Trax (menus, cruising, racing),
with the interactive score taking over for the *dramatic* moments (pursuits). This division — Trax for the norm,
the score for the drama — is `MusicFlow`'s core decision ([C66.1](01-music-runtime.md)).

## Why a licensed jukebox

Having a full *licensed-song jukebox* (rather than only original score) served the game commercially and
experientially:

- **Marketing.** The soundtrack was a *selling point* — the featured artists and songs were promoted, and "EA
  Trax" was a recognisable brand ([C58.2](../C58-Build-Pipeline/02-eagl-engine.md)). The music was part of the
  game's identity.
- **Player agency.** `TraxScreen`/`Jukebox` let players *choose* their soundtrack — enable/disable songs, pick
  favourites. The music is *personalizable*, like the car ([Chapter 56](../C56-Customization/C56-Customization.md)).
- **The cultural fit.** The mid-2000s tuner-culture context ([C56.4](../C56-Customization/04-visual.md)) came with
  a *soundtrack* — the licensed music *is* that culture, as much as the vinyls and body kits.

So EA Trax is the *cultural* music of Most Wanted — the licensed soundtrack that placed it in its moment, marketed
it, and let players curate it. It's a conventional but important system: a jukebox of great songs, the game's aural
brand. Combined with the interactive score ([C66.3](03-interactive-score.md)) for the dramatic moments, it gives MW
a soundtrack that's both *recognisable* (Trax) and *responsive* (the score) — the best of a licensed jukebox and a
dynamic score.

## RE implications

- **EA Trax** is the licensed-song jukebox — full songs, player-selectable (`TraxScreen`/`Jukebox`), init
  `TraxInit`.
- **It plays the non-tense contexts** — menus, free-roam, (some) races — the *default* music.
- **`MusicFlow` switches to the score** for pursuits ([C66.4](04-intensity-heat.md)) — Trax for the norm, the score
  for the drama.
- **A cultural/marketing feature** — the branded soundtrack, personalizable, fitting the tuner culture.

---

### Key takeaways

- **EA Trax** is Most Wanted's **licensed-song jukebox** — full branded songs (`Trax`/`TraxMusicMode`), selectable
  via `TraxScreen`/`Jukebox`, initialised by `TraxInit`.
- It plays the **non-tense contexts** — **menus, free-roam, and (some) races** — the *default* soundtrack.
- `MusicFlow` ([C66.1](01-music-runtime.md)) plays Trax in the norm and **switches to the interactive score** for
  pursuits ([C66.4](04-intensity-heat.md)) — **Trax for the norm, the score for the drama**.
- EA Trax is a **cultural and marketing feature** — a recognisable branded soundtrack, **player-curated**
  (enable/pick songs), fitting the mid-2000s tuner culture.
- With the interactive score, MW's music is both **recognisable** (Trax) and **responsive** (the score) — a
  licensed jukebox *and* a dynamic score.

**Continue:** [C66.3 — The interactive score](03-interactive-score.md) · [Chapter 66 hub](C66-Interactive-Music.md)
