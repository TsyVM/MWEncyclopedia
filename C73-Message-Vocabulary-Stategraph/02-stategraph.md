# C73.2 — The `M*` Stategraph

> **The one-sentence version:** the `MEnter*` messages *are* the game's flow states — `MEnterFreeRoam`,
> `MEnterSafeHouse`, `MEnterPostRaceFlow`, `MEnterRaceOverFlow`, `MEnteringGameplay` — and posting one transitions
> the game into that state, so the `M*` vocabulary is a **stategraph** driving the gameflow.

[← C73.1 — Messages vs events: the M/E split](01-m-vs-e.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md) ·
[Next: C73.3 — The `E*` event vocabulary →](03-e-events.md)

---

## The states are messages

The game is always in some *flow state* — free-roaming the city, in the safe house, running a race, watching the
post-race sequence. The striking thing about MW's design is that these states are **named as messages**: the
`MEnter*` family *is* the state list.

```
MEnteringGameplay     — entering active gameplay
MEnterFreeRoam        — free-roaming the open world
MEnterSafeHouse       — in the safe house (garage/customization)
MEnterPostRaceFlow    — the post-race sequence
MEnterRaceOverFlow    — the race-over handling
```

To *enter* a state, the game **posts the corresponding `MEnter*` message** ([C73.4](04-dispatch.md)). So the flow
isn't a hidden `switch` buried in C++ — it's a set of *named transitions* on the message bus, each a legible verb.
This is why the gameflow ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) is so readable: its
states announce themselves as messages.

> ✅ *Verified:* `MEnterFreeRoam`, `MEnterSafeHouse`, `MEnterPostRaceFlow`, `MEnterRaceOverFlow`, and
> `MEnteringGameplay` are strings in `speed.exe` — the flow-state entry messages.

## A graph of transitions

Because each state is an `MEnter*` message, the flow is a **graph**: nodes are states, edges are the transitions
between them. A typical traversal:

```
(boot) → MEnteringGameplay → MEnterFreeRoam ⇄ MEnterSafeHouse
                                   │
                              (start race)
                                   ▼
                            [racing gameplay]
                                   │
                              (finish)
                                   ▼
                         MEnterRaceOverFlow → MEnterPostRaceFlow → MEnterFreeRoam
```

Free-roam and the safe house cycle back and forth; starting a race leaves free-roam for gameplay; finishing enters
the race-over then post-race flow, which returns to free-roam. Loading punctuates transitions — `MLoad` /
`MLoadingComplete` bracket the streaming ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md))
between states. This graph *is* the "stategraph" of the chapter title: the map of how the game moves between its
modes, expressed entirely in `M*` messages.

> 🟡 *Reasoned:* the specific transition graph (which state leads to which) is the natural reading of the `MEnter*`
> state names and the gameflow ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)); the exact edge set
> is per-flow RE. The `MEnter*` state messages and `MLoad`/`MLoadingComplete` are verified.

## In-state toggles

Not every `M*` is a state transition — some *configure the current state*. The `MSet*` family flips gameplay
toggles without changing the flow state:

```
MSetTrafficSpeed       — traffic tuning (Ch.61)
MSetCopsEnabled        — cops on/off (Ch.49)
MSetCopAutoSpawnMode   — cop spawn behaviour (Ch.49)
```

These are *within-state* adjustments — you're still in free-roam, but now cops are enabled, or traffic is faster. So
the `M*` vocabulary has two kinds: **`MEnter*`** (change state) and **`MSet*`** (configure state). Together they give
the flow both *structure* (the state graph) and *parameters* (the toggles), all through the same message channel. A
flow script ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) drives both: post `MEnterFreeRoam` to enter,
`MSetCopsEnabled` to arm the cops.

## The online sub-graph

The `MOnline*` family is a **stategraph of its own** — the online/multiplayer flow
([Chapter 74](../C74-Multiplayer-Online/C74-Multiplayer-Online.md)):

```
MOnline → MOnlineCreateGame / MOnlineQuickRace / MOnlineOptiMatch
        → MOnlineRanking → MOnlineRankingPersonal / MOnlineRankingsLeaderboard
        → MOnlineSelectCar → MOnlineTOS / MOnlineNews
```

This is a parallel flow for the online menus — lobbies, matchmaking, rankings — expressed in the same `M*` message
style. That the online flow uses the identical `MEnter*`/`MSet*`-style vocabulary shows the stategraph pattern is
*general*: the game's front-end, gameplay, and online modes are all state machines on the message bus. Reading the
`MOnline*` family is reading the online flow's shape ([Chapter 74](../C74-Multiplayer-Online/C74-Multiplayer-Online.md)),
even where the servers are gone.

## RE implications

- **States are messages** — `MEnter*` *is* the flow-state list; posting one transitions the game.
- **A graph of transitions** — nodes are states, edges are `MEnter*` posts; `MLoad`/`MLoadingComplete` bracket the
  streaming between them.
- **In-state toggles** — `MSet*` configures the current state (cops, traffic) without changing it.
- **The online sub-graph** — `MOnline*` is a parallel stategraph for multiplayer
  ([Chapter 74](../C74-Multiplayer-Online/C74-Multiplayer-Online.md)).

---

### Key takeaways

- The `MEnter*` messages **are the flow states** — `MEnteringGameplay`, `MEnterFreeRoam`, `MEnterSafeHouse`,
  `MEnterPostRaceFlow`, `MEnterRaceOverFlow` — and **posting one transitions** the game, so the flow is a set of
  **named transitions on the message bus**, not a hidden `switch`.
- The `M*` vocabulary forms a **stategraph** — nodes (states) and edges (transitions) — that *is* the gameflow
  ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)); `MLoad`/`MLoadingComplete` bracket the
  streaming between states.
- Two kinds of `M*`: **`MEnter*`** (change state) and **`MSet*`** (configure the current state — cops, traffic) —
  giving the flow both **structure** and **parameters** through one channel.
- The **`MOnline*`** family is a **parallel stategraph** for the online flow
  ([Chapter 74](../C74-Multiplayer-Online/C74-Multiplayer-Online.md)) — the same pattern applied to multiplayer menus.
- Verified: the `MEnter*` state messages, `MSet*` toggles, `MLoad`/`MLoadingComplete`, and the `MOnline*` family.

**Continue:** [C73.3 — The `E*` event vocabulary](03-e-events.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md)
