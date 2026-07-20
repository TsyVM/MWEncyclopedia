# C72.3 — `LuaPostOffice`: the Message Bridge

> **The one-sentence version:** `LuaPostOffice` bridges Lua to the engine's message system — a script can *post*
> engine messages (the `M*`/`E*` verbs) and *receive* them through handlers — so Lua flow logic drives and reacts to
> the C++ systems using the same message vocabulary the engine speaks internally.

[← C72.2 — `LuaRuntime` & the bytecode chunks](02-runtime-bytecode.md) · [Chapter 72 hub](C72-Lua-Scripting.md) ·
[Next: C72.4 — `LuaAttributes`: the attribute bridge →](04-attributes-bridge.md)

---

## Messages as mail

The engine's systems communicate by **messages** — named events (`M*`/`E*` verbs,
[Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) posted and delivered rather
than called directly. The **`LuaPostOffice`** is the *bridge* that lets Lua join this conversation. The metaphor is
apt: a post office *sends* and *receives* mail, and `LuaPostOffice` lets a script both **post** messages into the
engine and **receive** messages from it.

- **Post** — a script sends an engine message (`M*`/`E*`), triggering a C++ system to act: start a race, change a
  screen, play a cue.
- **Receive** — the engine delivers a message to a script handler, so Lua flow logic *reacts*: on "race finished," go
  to the results screen.

So `LuaPostOffice` is the *two-way radio* between the policy layer (Lua, [C72.1](01-embedded-vm.md)) and the mechanism
layer (C++): scripts don't call engine functions ad hoc — they *post and receive messages*, the same decoupled
channel the engine's own systems use.

> ✅ *Verified:* `LuaPostOffice` is a string in `speed.exe` — the Lua↔message-system bridge; the message vocabulary
> it carries is decoded in [Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md).

## Why a message bridge (not direct calls)

Binding Lua to the engine *through messages* rather than direct function calls is a deliberate, clean design:

- **Decoupling** — the script doesn't need to know *which* C++ object handles "start race"; it posts the message and
  whoever's listening acts. The engine and scripts stay loosely coupled
  ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)).
- **One vocabulary** — scripts speak the *same* `M*`/`E*` messages the C++ systems speak, so there's a single event
  language across the whole engine, not a separate script API to maintain.
- **Symmetry** — because messages flow both ways, a script is a *peer* on the message bus, not a subordinate: it can
  drive the engine and be driven by it through the identical channel.

This is why the front-end flow ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) works so cleanly as Lua: a menu
script *posts* "go to car lot" and *receives* "back pressed," and the C++ screen system reacts and notifies — all
through `LuaPostOffice`. The script is pure policy ([C72.1](01-embedded-vm.md)), expressed as messages.

> 🟡 *Reasoned:* the post/receive model and the decoupling rationale follow from the "post office" naming and the
> engine's message-based architecture ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md));
> the exact API a script uses to post/subscribe is bytecode-level detail. The `LuaPostOffice` bridge is verified.

## The bridge in the flow

Placing `LuaPostOffice` in the layer stack shows its role as the *action* channel:

```
Lua script (policy)  ──post──▶  LuaPostOffice  ──▶  message system (Ch.73)  ──▶  C++ system acts
C++ system            ──message──▶  message system  ──▶  LuaPostOffice  ──deliver──▶  Lua handler (reacts)
```

Where `LuaAttributes` ([C72.4](04-attributes-bridge.md)) is the script's *read* channel (query the vault),
`LuaPostOffice` is its *act* channel (send/receive events). Together they're the script's two ways of touching the
engine: *know* the state (attributes) and *change/observe* it (messages). A script's logic is typically: receive a
message, read some attributes, decide, post a message — the loop of a policy layer, wired through these two bridges.

## RE implications

- **`LuaPostOffice` is the message bridge** — scripts **post** engine messages (`M*`/`E*`) and **receive** them via
  handlers.
- **Messages, not direct calls** — decoupling, one shared event vocabulary, symmetric (script as a peer on the bus).
- **The act channel** — `LuaPostOffice` (send/observe events) complements `LuaAttributes`
  ([C72.4](04-attributes-bridge.md)) (read state).
- **Powers the front-end flow** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) — menu scripts post/receive
  transitions.

---

### Key takeaways

- **`LuaPostOffice`** bridges Lua to the engine's **message system**
  ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) — a script can **post**
  engine messages and **receive** them through handlers.
- Binding **through messages, not direct calls** keeps the engine and scripts **decoupled**, gives them **one shared
  event vocabulary**, and makes a script a **peer** on the message bus (it drives and is driven symmetrically).
- It's the script's **act channel** — send/observe events — paired with `LuaAttributes`
  ([C72.4](04-attributes-bridge.md)), the **read channel** (query the vault); together, a script *knows* and *changes*
  the engine.
- This is why **front-end flow** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) is clean Lua — menu scripts
  post "go here" and receive "button pressed," expressing pure policy as messages.
- Verified: the `LuaPostOffice` bridge; its message vocabulary is
  [Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md).

**Continue:** [C72.4 — `LuaAttributes`: the attribute bridge](04-attributes-bridge.md) · [Chapter 72 hub](C72-Lua-Scripting.md)
