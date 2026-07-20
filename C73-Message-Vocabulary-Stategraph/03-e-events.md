# C73.3 — The `E*` Event Vocabulary

> **The one-sentence version:** the `E*` events are the engine's concrete gameplay vocabulary, grouped by verb —
> `EShow*` (UI screens), `ESpawn*` (world objects), `EPlay*`/`ETrigger*` (cinematics & effects), `EPlayer*` (player
> events), and `EReset*`/`ESet*` (reset & configure) — the actions the C++ systems actually perform.

[← C73.2 — The `M*` stategraph](02-stategraph.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md) ·
[Next: C73.4 — `GEventTable`: the dispatch →](04-dispatch.md)

---

## Events grouped by verb

Where the `M*` messages are flow ([C73.2](02-stategraph.md)), the `E*` events are *action* — and they organise
cleanly by their leading verb, each family a category of thing the engine does:

| Family | Does | Examples | Decoded in |
|---|---|---|---|
| `EShow*` | show a UI screen | `EShowResults`, `EShowRaceCountdown`, `EShowMilestones`, `EShowMessageScreen` | [Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md) |
| `ESpawn*` | spawn a world object | `ESpawnSmackable`, `ESpawnExplosion`, `ESpawnFragment` | [Ch. 52](../C52-Effects-Particles/C52-Effects-Particles.md)/[63](../C63-Collision-World/C63-Collision-World.md) |
| `EPlay*` / `ETrigger*` | play a cinematic/effect | `EPlayRaceNIS`, `EPlayEndNIS`, `EPlayRaceMovie`, `ETriggerMomentNIS`, `EPlayObjectEffect` | [Chapters 24–25](../C24-NIS-Animation/C24-NIS-Animation.md) |
| `EPlayer*` | a player event | `EPlayerTriggeredNOS`, `EPlayerShift`, `EPlayerAirborne` | [Chapters 39–42](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md) |
| `EReset*` | reset state | `EResetPlayerCar`, `EResetProps`, `EResetSystem`, `EResetSequencer` | — |
| `ESet*` | configure a system | `ESetSimRate`, `ESetCopAutoSpawnMode`, `ESetPlayerCarReset` | [Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md) |

So the `E*` vocabulary is a *taxonomy of engine actions*: show, spawn, play, notify, reset, set. Reading an event's
verb tells you its category, and its suffix the specific action — `EShowResults` shows the results screen,
`ESpawnExplosion` spawns an explosion. The naming is self-documenting ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)),
so the event list is a readable catalogue of what the engine can be asked to do.

> ✅ *Verified:* the `E*` families are strings in `speed.exe` — `EShow*` (`EShowResults`/`EShowRaceCountdown`/
> `EShowMilestones`/`EShowMessageScreen`/`EShowMarketingScreen`/`EShowSMS`/`EShowTimeExtension`), `ESpawn*`
> (`ESpawnSmackable`/`ESpawnExplosion`/`ESpawnFragment`), `EPlay*`/`ETrigger*` (`EPlayRaceNIS`/`EPlayEndNIS`/
> `EPlayRaceMovie`/`ETriggerMomentNIS`/`EPlayObjectEffect`), `EPlayer*` (`EPlayerTriggeredNOS`/`EPlayerShift`/
> `EPlayerAirborne`), `EReset*`, `ESet*`.

## Events cross the whole engine

The `E*` events are the *wiring* between systems — each event lets one system ask another to act, without a direct
call:

- **`EShow*` → the front-end** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)): gameplay posts `EShowResults`,
  the FEng shows the results screen. The race system doesn't call the UI — it posts an event.
- **`ESpawn*` → effects/objects** ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)): a collision
  posts `ESpawnExplosion`, the FX system spawns it. The collision code doesn't own particles — it posts an event.
- **`EPlay*` → the NIS/cinematic system** ([Chapters 24–25](../C24-NIS-Animation/C24-NIS-Animation.md)): the race
  end posts `EPlayEndNIS`, the cutscene system plays it.
- **`EPlayer*` → listeners** ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)): the sim posts `EPlayerAirborne`
  or `EPlayerTriggeredNOS`, and whoever cares (the score, the audio, the pursuit) reacts.

So the `E*` vocabulary is how the engine's systems *collaborate* — the render, physics, audio, UI, and cinematic
systems don't reference each other; they post and handle events. This is the decoupling
([C73.4](04-dispatch.md)) that lets a big engine stay modular: each system exposes what it *does* as events others can
trigger, and what it *reacts to* as events it handles.

## Events as gameplay's seams

The `EPlayer*` events are especially telling — they're the *sim broadcasting its state* for others to use:

- `EPlayerShift` — the player changed gear ([C40.4](../C40-Eight-Mechanics/04-engine.md)); the audio plays a shift
  sound, the HUD flashes the gear.
- `EPlayerTriggeredNOS` — nitrous fired ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)); the FX
  spawns the flames, the sim applies the boost, the HUD drains the meter.
- `EPlayerAirborne` — the car left the ground; the camera, the score ("air time"), and the physics react.

One player action, *many* reactions — because the sim *posts an event* and every interested system handles it, rather
than the sim calling each. This is the seam between *gameplay* (the sim) and *presentation* (audio, UI, FX, camera):
the sim announces, the presentation reacts, all through `E*` events. Reading them reveals how a single moment (a
shift, a jump, a NOS hit) fans out into the multi-system response the player experiences
([C66.4](../C66-Interactive-Music/04-intensity-heat.md)).

## RE implications

- **Grouped by verb** — `EShow`/`ESpawn`/`EPlay`/`ETrigger`/`EPlayer`/`EReset`/`ESet` — a taxonomy of engine actions.
- **Cross-engine wiring** — `EShow*`→UI, `ESpawn*`→FX, `EPlay*`→cinematics, `EPlayer*`→listeners — systems
  collaborate via events, not direct calls.
- **Gameplay's seams** — `EPlayer*` events broadcast the sim's state; one action fans out to many reactions.
- **Self-documenting** — verb = category, suffix = action; the event list is a readable action catalogue.

---

### Key takeaways

- The `E*` events are a **taxonomy of engine actions**, grouped by verb: **`EShow*`** (UI screens), **`ESpawn*`**
  (world objects), **`EPlay*`/`ETrigger*`** (cinematics/effects), **`EPlayer*`** (player events), **`EReset*`**
  (reset), **`ESet*`** (configure).
- Events are the engine's **cross-system wiring** — `EShowResults`→the FEng, `ESpawnExplosion`→the FX system,
  `EPlayEndNIS`→the cutscene system — so systems **collaborate without direct calls** (decoupling,
  [C73.4](04-dispatch.md)).
- **`EPlayer*` events are gameplay's seams** — the sim **broadcasts** its state (shift, NOS, airborne) and **many
  systems react** (audio, UI, FX, camera, score) — one action, a fanned-out response.
- The naming is **self-documenting** — verb = category, suffix = specific action — so the `E*` list is a **readable
  catalogue** of what the engine can be asked to do.
- Verified: the `EShow`/`ESpawn`/`EPlay`/`ETrigger`/`EPlayer`/`EReset`/`ESet` families in `speed.exe`.

**Continue:** [C73.4 — `GEventTable`: the dispatch](04-dispatch.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md)
