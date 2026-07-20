# C66.1 — The Music Runtime

> **The one-sentence version:** `MusicFlow` is the music state machine that selects what plays when — managing two
> systems, the EA Trax licensed jukebox and the dynamic interactive score — each with its own volume, both on the
> `PFEATrax` bus.

[← Chapter 66 hub](C66-Interactive-Music.md) · [Next: C66.2 — EA Trax →](02-ea-trax.md)

---

## MusicFlow: the state machine

**`MusicFlow`** is the music runtime's *conductor* — a state machine that decides *what music plays* in each game
context ([C54.1](../C54-GameFlow-Blacklist/01-gameflow-states.md)):

- **Menu** — the front-end music ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) — EA Trax
  ([C66.2](02-ea-trax.md)).
- **Free-roam** — ambient Trax songs.
- **Race** — race music (Trax or score).
- **Pursuit** — the interactive score ([C66.3](03-interactive-score.md)), intensifying with Heat
  ([C66.4](04-intensity-heat.md)).

So `MusicFlow` maps *game state* to *music state* — as you move between menu, free-roam, race, and pursuit
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)), it transitions the music accordingly. It's the
music counterpart of the GameFlow state machine ([C54.1](../C54-GameFlow-Blacklist/01-gameflow-states.md)) — one
drives the game's phase, the other the music's.

> ✅ *Verified:* `MusicFlow`, `MusicMode`, `MusicVol` are present in `speed.exe` — the music state machine and its
> mode/volume; `Trax`/`TraxMusicMode` and `Interactive`/`InteractiveMusicVol` are the two systems it manages.

## Two music systems

`MusicFlow` manages **two distinct music systems** ([above](#musicflow-the-state-machine)), each with its own
volume:

| System | What | Volume |
|---|---|---|
| **EA Trax** ([C66.2](02-ea-trax.md)) | licensed songs — the jukebox | `MusicVol` |
| **Interactive score** ([C66.3](03-interactive-score.md)) | dynamic gameplay music | `InteractiveMusicVol` |

That there are *separate volumes* (`MusicVol` vs. `InteractiveMusicVol`) confirms these are *independent* systems —
the licensed Trax and the dynamic score can be mixed separately (or one favoured over the other per context). So the
music runtime isn't one player — it's *two*, coordinated by `MusicFlow`: the *branded* jukebox (Trax) and the
*adaptive* score. Different contexts favour different systems: menus and free-roam lean on Trax (full songs),
pursuits on the interactive score (dynamic tension). The two-system design lets MW have *both* a great licensed
soundtrack *and* responsive gameplay music.

## Both on the PFEATrax bus

Both music systems route through the **`PFEATrax`** bus ([C59.1](../C59-Audio-Runtime/01-audio-runtime.md)) — the
music mix group in the audio runtime ([Chapter 59](../C59-Audio-Runtime/C59-Audio-Runtime.md)):

- **A dedicated music bus** — `PFEATrax` ([C59.1](../C59-Audio-Runtime/01-audio-runtime.md)) is the `SFXObj_*` bus
  for music, separate from ambience, collision, speech.
- **Independent mixing** — music can be balanced against the other buses ([C59.1](../C59-Audio-Runtime/01-audio-runtime.md))
  — e.g. ducked under speech ([C59.1](../C59-Audio-Runtime/01-audio-runtime.md)) or a callout.
- **The volumes** — `MusicVol`/`InteractiveMusicVol` set the levels *within* the music bus.

So the music runtime plugs into the audio engine ([Chapter 59](../C59-Audio-Runtime/C59-Audio-Runtime.md)) as the
`PFEATrax` bus — the two music systems ([above](#two-music-systems)) mixed there, then routed to the master with the
other buses. This is the audio-runtime integration ([Chapter 59](../C59-Audio-Runtime/C59-Audio-Runtime.md)): music
is one bus among many, coordinated by `MusicFlow` on top and mixed by the audio engine below. Reading the music
runtime is reading `MusicFlow` (the selection) + `PFEATrax` (the mixing) — what plays, and how loud.

## RE implications

- **`MusicFlow`** is the music state machine — maps game state (menu/free-roam/race/pursuit) to music state.
- **Two systems** — EA Trax (licensed, `MusicVol`) and the interactive score (dynamic, `InteractiveMusicVol`) —
  independent volumes.
- **Both on `PFEATrax`** — the music bus in the audio runtime, mixed against the other buses.
- **`MusicFlow` selects, `PFEATrax` mixes** — what plays, and how loud.

---

### Key takeaways

- **`MusicFlow`** is the music **state machine** — it maps game state (menu → free-roam → race → pursuit) to music
  state, the music counterpart of GameFlow.
- It manages **two independent systems** — **EA Trax** (licensed jukebox, `MusicVol`) and the **interactive score**
  (dynamic music, `InteractiveMusicVol`) — the *separate volumes* prove they're distinct.
- Different contexts favour different systems — **Trax** for menus/free-roam (full songs), the **interactive
  score** for pursuits (dynamic tension).
- Both route through the **`PFEATrax`** music bus ([C59.1](../C59-Audio-Runtime/01-audio-runtime.md)) — mixed
  against the other audio buses (duckable under speech).
- Reading the music runtime is **`MusicFlow` (selection) + `PFEATrax` (mixing)** — what plays and how loud.

**Continue:** [C66.2 — EA Trax](02-ea-trax.md) · [Chapter 66 hub](C66-Interactive-Music.md)
