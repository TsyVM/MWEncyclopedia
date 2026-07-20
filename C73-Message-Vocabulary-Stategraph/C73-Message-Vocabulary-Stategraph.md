# Chapter 73 — The Message Vocabulary & Stategraph

> **Goal of this chapter:** decode the engine's event language — the two-level vocabulary of **`M*` flow messages**
> (the stategraph: `MEnterFreeRoam`, `MEnterSafeHouse`, `MEnterPostRaceFlow`, `MSet*` toggles, `MOnline*` menus) and
> **`E*` engine events** (`EShow*`, `ESpawn*`, `EPlay*`, `EReset*`, `EPlayer*`, `ETrigger*`), and the `GEventTable`
> dispatch (`Post` at `0x65FAF0`) that delivers them.

The engine's systems don't call each other directly — they **post and receive messages**. This chapter decodes that
language: the `M*` messages that drive *flow* (the "stategraph" of game states) and the `E*` events that drive
*gameplay*, plus the `GEventTable` that dispatches both. It's the vocabulary the Lua layer speaks through
`LuaPostOffice` ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) and the wiring under the gameflow
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) — the engine's nervous system.

> **Verified against the executable.** The dispatch is `GEventTable::Post` at `0x65FAF0` (a thiscall method);
> `EventHandler`, `EVENT_*`, and 32 `Message` strings are present. The **`M*` flow vocabulary** includes
> `MEnterFreeRoam`, `MEnterSafeHouse`, `MEnterPostRaceFlow`, `MEnterRaceOverFlow`, `MEnteringGameplay`,
> `MLoadingComplete`, `MSetTrafficSpeed`, `MSetCopsEnabled`, `MSetCopAutoSpawnMode`, and the `MOnline*` family. The
> **`E*` engine vocabulary** includes `EShowResults`/`EShowRaceCountdown`/`EShowMilestones` (UI), `ESpawnSmackable`/
> `ESpawnExplosion`/`ESpawnFragment` (objects), `EPlayRaceNIS`/`EPlayEndNIS`/`ETriggerMomentNIS` (cinematics),
> `EPlayerTriggeredNOS`/`EPlayerShift`/`EPlayerAirborne` (player), `EResetPlayerCar`/`EResetProps` (reset),
> `ESetSimRate`/`ESetCopAutoSpawnMode` (set). Note the **`M`→`E` pairs** (`MSetCopAutoSpawnMode`→`ESetCopAutoSpawnMode`).

---

## Deep-dive pages

- [C73.1 — Messages vs events: the `M`/`E` split](01-m-vs-e.md): two levels — flow messages and engine events — and
  why they pair.
- [C73.2 — The `M*` stategraph](02-stategraph.md): the flow states (`MEnter*`) and the state machine they form.
- [C73.3 — The `E*` event vocabulary](03-e-events.md): the gameplay-event families (`EShow`/`ESpawn`/`EPlay`/…).
- [C73.4 — `GEventTable`: the dispatch](04-dispatch.md): how a posted message finds its handlers (`Post` at
  `0x65FAF0`).
- [C73.5 — Reading messages in RE](05-reading-messages.md): navigating the vocabulary from the strings.

---

## 73.1 Messages vs events: the M/E split

The vocabulary has **two levels** ([C73.1](01-m-vs-e.md)): **`M*`** are *flow messages* — high-level policy
(`MEnterFreeRoam`, `MSetCopsEnabled`), the language of the gameflow ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md))
and the Lua layer ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)); **`E*`** are *engine events* — lower-level
mechanism (`ESpawnExplosion`, `EPlayRaceNIS`), what the C++ systems actually do. The **`M`→`E` pairs**
(`MSetCopAutoSpawnMode`→`ESetCopAutoSpawnMode`) show the levels connect: a flow message translates to an engine event.

## 73.2 The M* stategraph

The `M*` messages form a **stategraph** ([C73.2](02-stategraph.md)) — a state machine of game flow. The `MEnter*`
messages *are* the states: `MEnterFreeRoam`, `MEnterSafeHouse`, `MEnterPostRaceFlow`, `MEnterRaceOverFlow`,
`MEnteringGameplay`. Posting an `MEnter*` transitions the game into that flow state, driving the gameflow
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) — the map of how the game moves between free-roam,
racing, the safe house, and the post-race sequence.

## 73.3 The E* event vocabulary

The `E*` events are the **gameplay vocabulary** ([C73.3](03-e-events.md)), grouped by verb: `EShow*` (show a UI
screen, [Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)), `ESpawn*` (spawn an object — smackable, explosion,
fragment), `EPlay*`/`ETrigger*` (play a cinematic or effect, [Chapters 24–25](../C24-NIS-Animation/C24-NIS-Animation.md)),
`EPlayer*` (player events — NOS, shift, airborne), `EReset*`/`ESet*` (reset or configure). These are the concrete
things the engine *does*.

## 73.4 `GEventTable`: the dispatch

Both `M*` and `E*` are delivered by the **`GEventTable`** ([C73.4](04-dispatch.md)) — `Post` at `0x65FAF0` posts a
message into the table, which routes it to the registered `EventHandler`s. This is the decoupling
([Chapter 72](../C72-Lua-Scripting/03-postoffice.md)): a poster names a message, and the table delivers it to whoever
listens — the poster and handler never reference each other directly.

---

### Key takeaways

- The engine's systems communicate by **posting and receiving messages**, in a **two-level vocabulary**: **`M*`
  flow messages** (policy) and **`E*` engine events** (mechanism), connected by **`M`→`E` pairs**.
- The **`M*` messages form a stategraph** — `MEnter*` messages *are* the flow states (free-roam, safe house,
  post-race), driving the gameflow ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).
- The **`E*` events** are the gameplay vocabulary — `EShow`/`ESpawn`/`EPlay`/`ETrigger`/`EPlayer`/`EReset`/`ESet` —
  the concrete things the engine does.
- Both are dispatched by **`GEventTable`** (`Post` at `0x65FAF0`) to registered `EventHandler`s — decoupled delivery
  ([Chapter 72](../C72-Lua-Scripting/03-postoffice.md)).
- This is the language the **Lua layer** speaks ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) and the
  wiring under the **gameflow** ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) — the engine's
  nervous system.

**Next:** [C73.1 — Messages vs events: the `M`/`E` split](01-m-vs-e.md).
