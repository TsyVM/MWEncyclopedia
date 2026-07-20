# C73.5 — Reading Messages in RE

> **The one-sentence version:** navigate the engine's event language by the `M*` flow vocabulary (the stategraph),
> the `E*` engine vocabulary (the action taxonomy), and `GEventTable::Post` (`0x65FAF0`, the dispatch) — reading the
> message list as a self-documenting map of what the game's flow *is* and what its engine *does*.

[← C73.4 — `GEventTable`: the dispatch](04-dispatch.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md) ·
[Next: Chapter 74 — Multiplayer & Online Services →](../C74-Multiplayer-Online/C74-Multiplayer-Online.md)

---

## Anchors for message RE

The event system is anchored on verified strings:

- **Flow messages** — `MEnter*` (the stategraph, [C73.2](02-stategraph.md)), `MSet*` (toggles), `MOnline*` (the
  online flow), `MLoad`/`MLoadingComplete`.
- **Engine events** — the `E*` families ([C73.3](03-e-events.md)): `EShow`/`ESpawn`/`EPlay`/`ETrigger`/`EPlayer`/
  `EReset`/`ESet`.
- **The dispatch** — `GEventTable::Post` at `0x65FAF0`, `EventHandler`, `EVENT_*` ([C73.4](04-dispatch.md)).

From these, the whole event system is navigable: the flow states, the engine actions, and how both are delivered.

## The RE workflow

Reading the message system:

1. **Enumerate the vocabulary** — grep the `M*` and `E*` strings ([C73.1](01-m-vs-e.md)); the flow and engine
   messages.
2. **Map the stategraph** — the `MEnter*` states and their transitions ([C73.2](02-stategraph.md)).
3. **Group the events** — the `E*` families by verb ([C73.3](03-e-events.md)); the action taxonomy.
4. **Trace the dispatch** — `GEventTable::Post` ([C73.4](04-dispatch.md)); how a message reaches its handlers.

The output is the full event picture: the flow (a stategraph of `M*`), the actions (a taxonomy of `E*`), and the
dispatch (`GEventTable`) that connects posters to handlers.

## The vocabulary is a map of the engine

The deepest use of the message list in RE is as a **map of the engine's capabilities** — because every message
*names something the engine can do or be*:

- **The `M*` list** is the flow's *state space* ([C73.2](02-stategraph.md)) — every mode the game can be in
  (free-roam, safe house, post-race, online) is an `MEnter*`. Read the `M*` list and you have the gameflow's shape
  ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)).
- **The `E*` list** is the engine's *action space* ([C73.3](03-e-events.md)) — every concrete thing it can be asked
  to do (show a screen, spawn an explosion, play a NIS) is an `E*`. Read the `E*` list and you have a catalogue of
  the engine's abilities.

So the message vocabulary is a *self-documenting index* of the engine ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)):
the states it can enter and the actions it can take, each named. For RE this is invaluable — before decompiling a
single function, the `M*`/`E*` strings tell you *what the engine's flow and capabilities are*, and then the handlers
(reached via `GEventTable`, [C73.4](04-dispatch.md)) tell you *how each is implemented*. The names are the table of
contents; the handlers are the chapters.

## The nervous system, whole

This chapter and the Lua layer ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) together decode the engine's
**nervous system**:

- **The vocabulary** (this chapter) — the messages: `M*` flow (stategraph) + `E*` engine (actions).
- **The dispatch** (this chapter) — `GEventTable`: the delivery.
- **The scripting** ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) — Lua posting/handling messages via
  `LuaPostOffice`, reading state via `LuaAttributes`.

Together they explain *how the engine's parts talk*: systems (C++ and Lua) post named messages, `GEventTable`
delivers them, and handlers react — a decoupled bus of collaborating parts. The gameflow
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) rides on the `M*` stategraph; the gameplay rides
on the `E*` events; the scripting ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) rides on both. Reading the
message system is reading the *connective tissue* that turns a pile of systems into one coordinated game.

## RE implications

- **Anchor on** the `M*`/`E*` families and `GEventTable::Post` (`0x65FAF0`).
- **The RE workflow** — enumerate vocabulary → map stategraph → group events → trace dispatch.
- **A map of the engine** — `M*` = the flow's state space, `E*` = the engine's action space; the names are the table
  of contents.
- **The nervous system** — with [Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md), the full account of how the
  engine's parts communicate.

---

### Key takeaways

- The event system is anchored on the **`M*`** flow vocabulary (the stategraph), the **`E*`** engine vocabulary (the
  action taxonomy), and **`GEventTable::Post`** (`0x65FAF0`, the dispatch).
- The RE workflow: **enumerate the vocabulary → map the `MEnter*` stategraph → group the `E*` families → trace
  `GEventTable`** — the flow, the actions, and their delivery.
- The vocabulary is a **self-documenting map of the engine** — the `M*` list is the flow's **state space**
  ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)), the `E*` list the engine's **action space** —
  the names are the **table of contents**, the handlers the chapters.
- With the Lua layer ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)), this decodes the engine's **nervous
  system** — how C++ and Lua systems **post, dispatch, and handle** named messages to act as one coordinated game.
- Verified: the `M*`/`E*` families and `GEventTable::Post` at `0x65FAF0`.

**Next:** [Chapter 74 — Multiplayer & Online Services](../C74-Multiplayer-Online/C74-Multiplayer-Online.md).

**Sources:** `speed.exe` (verified: `GEventTable::Post` @ `0x65FAF0`; `EventHandler`, `EVENT_*`; the `M*` family —
`MEnterFreeRoam`/`MEnterSafeHouse`/`MEnterPostRaceFlow`/`MEnterRaceOverFlow`/`MEnteringGameplay`, `MLoadingComplete`,
`MSetTrafficSpeed`/`MSetCopsEnabled`/`MSetCopAutoSpawnMode`, `MOnline*`; the `E*` family — `EShow*`, `ESpawn*`,
`EPlay*`/`ETrigger*`, `EPlayer*`, `EReset*`, `ESet*`; the `MSetCopAutoSpawnMode`→`ESetCopAutoSpawnMode` pair). The
Lua bridge is [Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md); the gameflow is
[Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md).
