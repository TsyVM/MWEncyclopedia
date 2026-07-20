# C72.4 — `LuaAttributes`: the Attribute Bridge

> **The one-sentence version:** `LuaAttributes` bridges Lua to the attribute vault — a script can read the same
> attribute values (tuning, gameplay parameters) the sim consumes — so the data-driven vault and the script-driven
> flow meet, letting scripts *reason about* the engine's own data.

[← C72.3 — `LuaPostOffice`: the message bridge](03-postoffice.md) · [Chapter 72 hub](C72-Lua-Scripting.md) ·
[Next: C72.5 — What's moddable & reading Lua in RE →](05-reading-lua.md)

---

## The read channel

Where `LuaPostOffice` ([C72.3](03-postoffice.md)) lets a script *act* (post/receive messages), **`LuaAttributes`**
lets a script *know* — it bridges Lua to the **attribute vault** ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)),
the game's data store of tuning and gameplay parameters. Through it, a script can **read attribute values** by their
key ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) — a car's stats, a race's parameters, a heat
threshold — and use them in its logic.

So a script isn't blind to the game's data — it can query the *same* vault the C++ sim reads
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). This is what makes scripted policy *data-aware*: the
front-end can display a car's real stats ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md))
because `LuaAttributes` hands the script the vault values; an event script can branch on a gameplay parameter because
it can read it.

> ✅ *Verified:* `LuaAttributes` is a string in `speed.exe` — the Lua↔vault bridge; the vault it reads is decoded in
> [Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md).

## One data store, two readers

`LuaAttributes` means the vault has *two* kinds of reader, and it matters that they share one source:

- **The C++ sim** reads the vault for *simulation* — torque, grip, mass ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
- **Lua scripts** read the vault (via `LuaAttributes`) for *policy* — display a stat, gate an event, drive the UI.

Because both read the *same* attributes ([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)), the script's
view and the sim's view can't disagree — the number the garage shows *is* the number the sim uses
([C69.4](../C69-Performance-Upgrades-Tuning/04-upgrade-to-behaviour.md)). This is the same single-source-of-truth
discipline the tuning bars rely on ([C69.3](../C69-Performance-Upgrades-Tuning/03-tuning-bars.md)), realised at the
scripting boundary: `LuaAttributes` gives the script the *authoritative* data, so scripted UI and simulated behaviour
stay consistent.

## Reading by reflection

The bridge works through the vault's **reflection** system ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)):
attributes are keyed by hashed names ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)), and
`LuaAttributes` resolves a script's request for a named attribute to its vault value. So a script asks for an
attribute *by name*, the bridge hashes and looks it up ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)), and
returns the resolved value. This is why the bridge is thin: it doesn't reimplement attribute storage, it *exposes the
existing reflection lookup* to Lua. The vault's design — self-describing, hash-keyed
([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) — is exactly what makes it cheap to bridge to a
scripting language: expose one "get attribute by name" function and the whole data store is scriptable.

> 🟡 *Reasoned:* that `LuaAttributes` reads through the reflection/hash lookup ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md))
> follows from the vault's hash-keyed design and the bridge's job; whether scripts can *write* attributes (vs.
> read-only) is bytecode-level detail ([C72.5](05-reading-lua.md)). The `LuaAttributes` bridge and its link to the
> vault are verified.

## The two bridges together

`LuaAttributes` (read) and `LuaPostOffice` (act, [C72.3](03-postoffice.md)) are the script's *complete* interface to
the engine — everything a policy layer needs:

```
know  →  LuaAttributes  →  read the vault (state)          [Ch.11–14]
act   →  LuaPostOffice   →  post/receive messages (events)  [Ch.73]
```

A typical script loop is: *receive* a message (something happened), *read* attributes (what's the state), *decide*
(policy), *post* a message (make something happen). That's the whole job of the scripting layer
([C72.1](01-embedded-vm.md)) — sense, decide, act — and these two bridges are its senses and hands. Everything else in
the engine is the C++ mechanism the script orchestrates through them. Reading the Lua layer means reading these two
bridges: what a script can *know* (attributes) and what it can *do* (messages).

## RE implications

- **`LuaAttributes` is the read channel** — scripts read vault attributes
  ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) by key.
- **One data store, two readers** — sim (simulation) and scripts (policy) read the same vault, so views can't
  disagree.
- **Through reflection** — attributes are hash-keyed ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md));
  the bridge exposes the existing lookup, so it's thin.
- **The two bridges** — `LuaAttributes` (know) + `LuaPostOffice` (act) = the script's complete engine interface.

---

### Key takeaways

- **`LuaAttributes`** is the script's **read channel** — it bridges Lua to the **vault**
  ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)), letting a script read the **same attributes the
  sim consumes** (tuning, gameplay params).
- The vault gains **two readers** — the C++ sim (simulation) and Lua scripts (policy) — sharing **one source**, so a
  scripted display **can't disagree** with the simulated behaviour ([C69.4](../C69-Performance-Upgrades-Tuning/04-upgrade-to-behaviour.md)).
- It reads **through reflection** ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) — attributes are
  hash-keyed, so the bridge just **exposes the existing lookup** (thin by design; the vault's self-describing shape
  makes it cheap to script).
- With `LuaPostOffice` ([C72.3](03-postoffice.md)), it forms the script's **complete interface** — **know** (attributes)
  + **act** (messages) — the *sense/decide/act* loop of a policy layer.
- Verified: the `LuaAttributes` bridge and its link to the vault
  ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)).

**Continue:** [C72.5 — What's moddable & reading Lua in RE](05-reading-lua.md) · [Chapter 72 hub](C72-Lua-Scripting.md)
