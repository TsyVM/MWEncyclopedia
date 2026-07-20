# C73.4 — `GEventTable`: the Dispatch

> **The one-sentence version:** both `M*` and `E*` are delivered by `GEventTable` — `Post` at `0x65FAF0` puts a
> message into the table, which routes it to the registered `EventHandler`s — a publish/subscribe dispatch where the
> poster names a message and never references who handles it.

[← C73.3 — The `E*` event vocabulary](03-e-events.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md) ·
[Next: C73.5 — Reading messages in RE →](05-reading-messages.md)

---

## The event table

Every message — `M*` flow or `E*` engine — flows through one place: the **`GEventTable`**, the engine's message
router. Its core operation is **`Post`**, at `0x65FAF0`: a `thiscall` method that takes a message and delivers it to
the handlers registered for it.

```
0x65FAF0  GEventTable::Post:
  56 57         push esi ; push edi
  8B F9         mov  edi, ecx        ; edi = this (the GEventTable)
  51            push ecx
  8B 0F         mov  ecx, [edi]      ; read a table member
  8B C4 89 08   ...                  ; marshal the message
  8B 57 04      mov  edx, [edi+4]    ; another member
  51 ...        push ...             ; dispatch
```

The `thiscall` shape (`ecx` = `this`, reading `[edi]`/`[edi+4]` members) confirms `GEventTable` is an *object* with
state — the registry of handlers and the message queue. `Post` is how anything gets a message into that machine.

> ✅ *Verified:* `GEventTable::Post` is at `0x65FAF0` (file offset `0x25FAF0`), a `thiscall` method with prologue
> `56 57 8B F9 51 8B 0F …` (`push esi/edi; mov edi,ecx`); `EventHandler` and `EVENT_*` are strings in `speed.exe`.

## Publish, subscribe, deliver

`GEventTable` implements the **publish/subscribe** pattern — the decoupling the whole message system rests on
([C73.1](01-m-vs-e.md)):

- **Subscribe** — a system registers an **`EventHandler`** for the messages it cares about ("tell me when
  `EShowResults` is posted").
- **Publish** — any system calls `Post` with a message ("`EShowResults`!") — *without knowing who's subscribed*.
- **Deliver** — `GEventTable` looks up the message's handlers and calls each.

So the poster and the handler never reference each other — they meet *only* through the table and the message name.
The race system posts `EShowResults` and doesn't know (or care) that the FEng ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md))
is the one that handles it; the FEng subscribed and doesn't know the race system is the one that posted. The
`GEventTable` is the *broker* between them. This is why the engine's systems are so modular
([C73.3](03-e-events.md)): the table is the one thing they all share, and the message name is the only contract.

> 🟡 *Reasoned:* the subscribe/publish/deliver model (a handler registry keyed by message, dispatched by `Post`)
> follows from the `GEventTable`/`EventHandler` naming and the `thiscall` object shape; the exact registry structure
> and lookup are per-function RE. The `Post` address, its `thiscall` shape, and the `EventHandler`/`EVENT_*` strings
> are verified.

## The bridge and the bus

`GEventTable` is what `LuaPostOffice` ([C72.3](../C72-Lua-Scripting/03-postoffice.md)) plugs into — the reason a
script can post and receive engine messages is that the *engine already routes everything through the table*, and the
bridge just exposes `Post`/subscribe to Lua:

```
Lua script → LuaPostOffice → GEventTable.Post → registered EventHandlers (C++ and/or Lua)
```

So the message bus is *shared* between C++ and Lua ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)): a script
and a C++ system are peers on the same `GEventTable`, posting and handling the same `M*`/`E*` messages. This is the
deepest reason the two-level vocabulary works ([C73.1](01-m-vs-e.md)) — there's *one* dispatch mechanism, and both
the scripted flow and the compiled engine speak to it. The stategraph ([C73.2](02-stategraph.md)) and the event
taxonomy ([C73.3](03-e-events.md)) are just *what* gets posted; `GEventTable` is *how* it's delivered.

## Why a table, not calls

Routing everything through `GEventTable` rather than direct function calls is the architectural choice that shapes the
engine:

- **Modularity** — systems depend on *messages*, not on each other; you can add a handler (a new system that reacts
  to `EPlayerAirborne`) without touching the poster.
- **Multiplicity** — one message, many handlers ([C73.3](03-e-events.md)): a `Post` fans out to every subscriber, so
  a single event drives a coordinated multi-system response.
- **Scriptability** — because delivery is table-mediated, Lua can join as a peer
  ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) — impossible if systems called each other directly.

The cost is indirection (a post is slower than a call, and the flow is harder to trace statically). MW accepts it
because the *flexibility* — modular systems, fan-out reactions, scriptable flow — is worth far more than the cycles,
for the *policy* layer where these messages live ([C73.1](01-m-vs-e.md)). Reading `GEventTable` is reading the choice
that makes the engine a *bus of collaborating systems* rather than a call graph.

## RE implications

- **`GEventTable` is the router** — `Post` at `0x65FAF0` (thiscall) delivers `M*`/`E*` messages to registered
  `EventHandler`s.
- **Publish/subscribe** — poster names a message, table delivers to subscribers; poster and handler never reference
  each other.
- **The shared bus** — `LuaPostOffice` ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)) plugs into the same
  table, so C++ and Lua are peers on it.
- **A table, not calls** — modularity, one-to-many fan-out, and scriptability, at the cost of indirection.

---

### Key takeaways

- Both `M*` and `E*` are delivered by **`GEventTable`** — **`Post`** at **`0x65FAF0`** (a `thiscall` method on the
  table object) routes a message to its registered **`EventHandler`s**.
- It's a **publish/subscribe** dispatch — a system **subscribes** a handler, any system **publishes** via `Post`
  *without knowing who listens*, and the table **delivers** — so posters and handlers meet **only through the message
  name**.
- `GEventTable` is the **shared bus** that `LuaPostOffice` ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md))
  plugs into — C++ systems and Lua scripts are **peers** posting/handling the same messages.
- Routing through a **table, not direct calls**, buys **modularity** (systems depend on messages), **fan-out** (one
  message, many handlers), and **scriptability** — at the cost of indirection, worth it for the policy layer.
- Verified: `GEventTable::Post` at `0x65FAF0` (thiscall) and the `EventHandler`/`EVENT_*` strings.

**Continue:** [C73.5 — Reading messages in RE](05-reading-messages.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md)
